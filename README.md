# L'Architecture AWS mis au point pour l'examen

## Composants

### Bucket S3
- **Nom** : `{EnvName}-file-metadata-bucket`
- **Configuration** : Déclencheur Lambda sur les événements de création d'objets
- **Fonction** : Stocke les fichiers qui déclencheront le traitement des métadonnées

### Table DynamoDB
- **Nom** : `FileMetadata-{EnvName}`
- **Structure simple** avec :
  - Clé de partition : `FileName` (String)
  - Capacité provisionnée : 5 unités de lecture/écriture

### Fonction Lambda
- **Nom** : `learn-function-{EnvName}`
- **Runtime** : Python 3.13
- **Déclenchée par** : les événements de création d'objets S3
- **Fonction** : Stocke les métadonnées des fichiers (nom et nom du bucket) dans DynamoDB
- **Rôle IAM** : `arn:aws:iam::629193321657:role/lambda-s3-trigger-role`

### Instance EC2
- **Type d'instance** : t2.micro
- **AMI** : ami-0160e8d70ebc43ee1
- **Groupe de sécurité** : Groupe personnalisé autorisant SSH depuis 102.64.171.53/24
- **Script User Data** :
  - Installe AWS CLI
  - Effectue un scan DynamoDB au déploiement
- **Stockage** :
  - Volume EBS io1 de 20GB (200 IOPS)
- **Tags** : Nom d'environnement et classe (IABIGDATAM1)

### Groupe de Sécurité
- **Nom** : `{EnvName}-security-group`
- **Autorise** : Accès SSH (port 22) depuis une plage IP spécifique

## Flux de Travail
1. Les fichiers uploadés dans le bucket S3 déclenchent la fonction Lambda
2. La Lambda traite l'événement et stocke les métadonnées dans DynamoDB
3. L'instance EC2 peut accéder aux métadonnées via des opérations de scan DynamoDB

## Paramètres de Déploiement
- `EnvName` : Identifiant d'environnement (par défaut : "dev")
- `VPcId` : ID VPC pour l'instance EC2 (valeur par défaut fournie)

## Dépendances
- Le bucket S3 dépend des permissions Lambda créées au préalable
- La fonction Lambda nécessite un rôle IAM existant
- L'instance EC2 utilise une paire de clés existante ("iabdkey") et un sous-réseau

## Notes de Sécurité
- La Lambda utilise un ARN de rôle en dur (devrait être paramétré en production)
- Accès SSH à l'EC2 restreint à une plage IP spécifique
- Envisager d'ajouter le chiffrement pour S3 et DynamoDB en production

- 

# **Prérequis Essentiels**  

1. **Compte AWS**  
   - Permissions pour gérer S3, Lambda, DynamoDB, EC2.  

2. **Rôle IAM pour Lambda**  
   - Avec politiques :  
     - `AmazonS3ReadOnlyAccess`  
     - `AmazonDynamoDBFullAccess`  
     - `AWSLambdaBasicExecutionRole`  

3. **Clé SSH pour EC2**  

4. **Réseau (VPC/Subnet)**  

5. **Limites AWS**  

**À modifier avant déploiement** :  
- IDs codés en dur (VPC, subnet, rôle IAM).  
- IP SSH (`102.64.171.53/24` → Adresse IP publique).  

> *Pour vérifier l'adresse IP :* `curl ifconfig.me`dans la ligne de commande ou aller sur le site internet `https://whatismyipaddress.com/`
