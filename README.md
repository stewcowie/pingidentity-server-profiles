See [server profiles](https://pingidentity-devops.gitbook.io/devops/config) docs on gitbook for more information.
#------------------------------------------------------------------------------------------
#- Ping Identity integrated demo
#-
#-     app    console  
#-    3000    9000     
#-      |      |       
#-   +----------------+
#-   |   PingAccess   |
#-   +----------------+
#-
#-     login  console            app                console           rest    ldaps
#-     9031   9999               9022                 8443            1443    1636
#-      |      |                  |                    |               |       |
#-   +---------------+    +---------------+    +---------------+    +---------------+
#-   | PingFederate  |    |  PingCentral  |    |PingDataConsole|    | PingDirectory |
#-   +---------------+    +---------------+    +---------------+    +---------------+
#-
#-   +-----------------------+------------------------------------------------------------+
#-   |  Product Console/App  |  URL                                                       |
#-   |                       |    username: administrator                                 |
#-   |                       |    password: 2FederateM0re                                 |
#-   +-----------------------+------------------------------------------------------------+
#-   |  PingAccess           |  https://ping.lloydstsb.com:9000/                          |
#-   |  PingFederate         |  https://ping.lloydstsb.com:9999/pingfederate/app          |
#-   |  PingDirectory        |  https://ping.lloydstsb.com:8443/   (Server=pingdirectory) |
#-   |  PingCentral          |  https://ping.lloydstsb.com:9022/                          |
#-   |  PingOne              |  https://https://admin.pingone.com/web-portal/login        |
#-   +-----------------------+------------------------------------------------------------+
#------------------------------------------------------------------------------------------

### Pre-Requisites
* [Docker](https://www.docker.com/get-started)
* [Docker-Compose](https://docs.docker.com/compose/install/)
* AD PingFederate read only service account
* AD SPN record configurated for Kerberos integration (https://support.pingidentity.com/s/article/setSPN-Configuration-for-IWA-Adapter)
* AD Managed Service Account configured for Kerberos Constrained Delegation (https://pingidentity.sharefile.com/d-s5eed0e5e2894354a)


### Deployment - Docker Compose
* Copy the `docker-compose.yaml`file to a folder
* Modify the Server Profile URL's/accounts/access tokens to point to the LGB private repo
* Launch the stack with `docker-compose up -d`
* Logs for the stack can be watched with `docker-compose logs -f`
* Logs for individual services can be watched with `docker-compose logs -f {service}`

### Administrator Console Access
* See URL's listed above
* Ensure have an approriate local host/DNS record configured

### Post Deployment
* The Following LBG AD/Kerberos settings require to be validated/ammended as appropriate:
- PingFederate:
1. AD LDAPS cert added: Security -> Trusted CAs
2. AD Data Store settings: System -> Data Stores -> AD -> LDAP Configuration
3. AD Password Credential Validator: System -> Password Credential Validators -> AD -> Instance Configuration
4. AD Domain/Kerberos realm: System -> Active Directory Domains/Kerberos Realms -> iaglobal.lloydstsb.com
5. Kerberos adapter: Identity Provider -> Adapters -> Kerberos -> IdP Adapter

- PingAccess:
1. Kerberos Constrained Delegation Site Authenticator: Sites -> Site Authenticators -> KCD
2. krb5.ini: Located in lbg-layered-profiles/extensions/pingaccess/instance/conf/krb5.ini (see https://pingidentity.sharefile.com/d-s5eed0e5e2894354a)

Note:  Dev time permitting and once confirmed, the final values can later be passed as variables in the docker-compose.yaml as opposed to manaully editing.  See https://pingidentity-devops.gitbook.io/devops/config/containeranatomy/profilessubstitution

### Updating Server Profiles
* Following any changes, the product configuration should be exported from the admin console UI and commited to the git repo to update the server profiles.
* For PingFederate:
1. Export config via System -> Configuration Archive -> Export
2. Unzip the exported archive
3. Rename to folder to "data" and copy to pingidentity-server-profiles⁩ ▸ ⁨lbg-layered-profiles⁩ ▸ ⁨config⁩ ▸ ⁨pingfederate⁩ ▸ ⁨instance⁩ ▸ ⁨server⁩ (replace existing data folder)
* For PingAccess:
1. Export config via Settings -> System -> Export Configuration
2. Rename the export to data.json and copy to ⁨lbg⁩ ▸ ⁨pingaccess⁩ ▸ ⁨instance ▸ ⁨data (replace existing data.json)

### Use cases
* The following applications have been connected to PingFederate to test end user SSO/authentication policies
#- PingOne:
https://ping.lloydstsb.com:9031/idp/startSSO.ping?PartnerSpId=http%3A%2F%2Fpingone.com%2F15bb0af3-2c6b-465d-a1a4-5c69ef5cb088

#- Dummy SP Connection (httpbin.org)
https://ping.lloydstsb.com:9031/idp/startSSO.ping?PartnerSpId=https%3A%2F%2Fhttpbin.org

#- OAuth Playground
https://ping.lloydstsb.com:9031/OAuthPlayground/welcome

#- PingAccess Quick Start
https://ping.lloydstsb.com:3000/PingAccessQuickStart/headers

#- PingAccess Quick Start with KCD Site Authenticator
https://ping.lloydstsb.com:3000/PingAccessQuickStartKCD/headers

* Connected AD user accounts can be used to login
* Authentication policy can be changed by modifying the 'Extended Properties' value on PingFederate connections and OAuth clients.  Available values are:
#- "L01-Basic" = Kerberos with fallback to HTML form
#- "L02-MFA" = Kerberos with fallback to HTML form + PingID MFA
