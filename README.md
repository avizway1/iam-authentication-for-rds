# iam-authentication-for-rds


1. Create a policy, that allow to connect to RDS
2. Create an IAM user with prog access
3. Login to RDS using admin cred and create a user to use IAM Auth.
4. Download a Ca-cert
5. We can generate a token
6. Connect as new IAM user using token.

```bash
CREATE USER avinash IDENTIFIED WITH AWSAuthenticationPlugin AS 'RDS';
GRANT ALL ON `%`.* TO avinash@`%`;
ALTER USER 'avinash' REQUIRE SSL;
FLUSH PRIVILEGES;
```

Descriptions for each of the commands:

**CREATE USER avinash IDENTIFIED WITH AWSAuthenticationPlugin AS 'RDS';**:
   - This command creates a new database user named 'avinash' with AWS authentication plugin enabled. This means that the user will authenticate with AWS credentials rather than a traditional password.
   
**GRANT ALL ON `%`.* TO avinash@`%`;**:
   - This command grants all privileges on all tables in all databases (`%`.* indicates all tables in all databases) to the user 'avinash' from any host (`%`). This essentially gives the user full access to the databases.

**ALTER USER 'avinash' REQUIRE SSL;**:
   - This command modifies the user 'avinash' to require SSL (Secure Sockets Layer) connections when connecting to the database server. This enhances the security of the connection by encrypting the data transmitted between the client and the server.

**FLUSH PRIVILEGES;**:
   - This command reloads the privileges from the grant tables in the MySQL database. It ensures that any changes made to the privileges (such as creating users or granting privileges) take effect immediately without needing to restart the MySQL server.



The certificate bundle contains the SSL certificates necessary to establish secure connections between client applications and the Amazon RDS database instances. These certificates are used to encrypt the data transmitted over the network, ensuring confidentiality and integrity of the information exchanged between the client and the RDS instance.

```bash
Download the certificate bundle : https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html#UsingWithRDS.SSL.CertificatesAllRegions
```

below command will generate an authentication token that can be used by the specified user (avinash) to authenticate with the RDS instance specified by the hostname, port, and region options. This token is typically used in scenarios where traditional username/password authentication is not feasible or less secure, such as when using IAM (Identity and Access Management) authentication with RDS.


```bash
aws rds generate-db-auth-token --hostname rdsauth.cluster-cfpgnjehw330.ap-south-1.rds.amazonaws.com --port 3306 --region ap-south-1 --username avinash
```

```bash
mysql --host=rdsauth.cluster-cfpgnjehw330.ap-south-1.rds.amazonaws.com --port=3306 --ssl-ca=global-bundle.pem --user=avinash --password='rdsauth.cluster-cfpgnjehw330.ap-south-1.rds.amazonaws.com:3306/?Action=connect&DBUser=avinash&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Expires=900&X-Amz-Credential=AKIAXJMA6A45YB4W3LLZ%2F20240419%2Fap-south-1%2Frds-db%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Date=20240419T165343Z&X-Amz-Signature=68dc620cbefa974016eec324f747b805aa995b3d178fd2705ab498ceb4ef2f89'
```

Below command sets up variables to facilitate the secure connection to an Amazon RDS (Relational Database Service) instance using IAM (Identity and Access Management) authentication.

```bash
HOST='rdsauth.cluster-cfpgnjehw330.ap-south-1.rds.amazonaws.com'
```

```bash
TOKEN=$(aws rds generate-db-auth-token --hostname $HOST --port 3306 --region ap-south-1 --username avinash)
```

```bash
mysql --host=$HOST --port=3306 --ssl-ca=global-bundle.pem --user=avinash --password=$TOKEN
```
