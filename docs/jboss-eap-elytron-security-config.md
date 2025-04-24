## Create a FIleSystemRealm

/subsystem=elytron/filesystem-realm=exampleSecurityRealm:add(path=fs-realm-users,relative-to=jboss.server.config.dir)

Add a User:

/subsystem=elytron/filesystem-realm=exampleSecurityRealm:add-identity(identity=user1)

Set Password:
/subsystem=elytron/filesystem-realm=exampleSecurityRealm:set-password(identity=user1, clear={password="passwordUser1"})

# roles
/subsystem=elytron/filesystem-realm=exampleSecurityRealm:add-identity-attribute(identity=user1, name=Roles, value=["Admin","Guest"])
{"outcome" => "success"}
Security Domain which references the filesystem-realm:
/subsystem=elytron/security-domain=exampleSecurityDomain:add(default-realm=exampleSecurityRealm,permission-mapper=default-permission-mapper,realms=[{realm=exampleSecurityRealm}])

Verify:
/subsystem=elytron/security-domain=exampleSecurityDomain:read-identity(name=user1)
{
    "outcome" => "success",
    "result" => {
        "name" => "user1",
        "attributes" => {"Roles" => [
            "Admin",
            "Guest"
        ]},
        "roles" => [
            "Guest",
            "Admin"
        ]
    }
}

Http Authentication Factory:
/subsystem=elytron/http-authentication-factory=myHttpAuthFactory:add(http-server-mechanism-factory=global, security-domain=exampleSecurityDomain, mechanism-configurations=[{mechanism-name="BASIC", realm-name="exampleSecurityRealm"}])

/subsystem=undertow/application-security-domain=myWebAppDomain:add(http-authentication-factory=myHttpAuthFactory)

### Creating an encrypted filesystem-realm in Elytron 

/subsystem=elytron/secret-key-credential-store=examplePropertiesCredentialStore:add(path=examplePropertiesCredentialStore.cs, relative-to=jboss.server.config.dir)

/subsystem=elytron/filesystem-realm=exampleSecurityRealm:add(path=fs-realm-users,relative-to=jboss.server.config.dir, credential-store=examplePropertiesCredentialStore, secret-key=key)

/subsystem=elytron/filesystem-realm=exampleSecurityRealm:add-identity(identity=user1)
{"outcome" => "success"}
/subsystem=elytron/filesystem-realm=exampleSecurityRealm:set-password(identity=user1, clear={password="passwordUser1"})
{"outcome" => "success"}
/subsystem=elytron/filesystem-realm=exampleSecurityRealm:add-identity-attribute(identity=user1, name=Roles, value=["Admin","Guest"])
{"outcome" => "success"}

/subsystem=elytron/security-domain=exampleSecurityDomain:add(default-realm=exampleSecurityRealm,permission-mapper=default-permission-mapper,realms=[{realm=exampleSecurityRealm}])
{"outcome" => "success"}



## LDAP Realm with Elytron

Sample LDIF file
```sh
dn: ou=Users,dc=wildfly,dc=org
objectClass: organizationalUnit
objectClass: top
ou: Users

dn: uid=user1,ou=Users,dc=wildfly,dc=org
objectClass: top
objectClass: person
objectClass: inetOrgPerson
cn: user1
sn: user1
uid: user1
userPassword: userPassword1

dn: ou=Roles,dc=wildfly,dc=org
objectclass: top
objectclass: organizationalUnit
ou: Roles

dn: cn=Admin,ou=Roles,dc=wildfly,dc=org
objectClass: top
objectClass: groupOfNames
cn: Admin
member: uid=user1,ou=Users,dc=wildfly,dc=org

```

/subsystem=elytron/dir-context=exampleDirContext:add(url="ldap://10.88.0.2",principal="cn=admin,dc=wildfly,dc=org",credential-reference={clear-text="secret"})

/subsystem=elytron/ldap-realm=exampleSecurityRealm:add(dir-context=exampleDirContext,identity-mapping={search-base-dn="ou=Users,dc=wildfly,dc=org",rdn-identifier="uid",user-password-mapper={from="userPassword"},attribute-mapping=[{filter-base-dn="ou=Roles,dc=wildfly,dc=org",filter="(&(objectClass=groupOfNames)(member={1}))",from="cn",to="Roles"}]})


The above commands results into the following xml fragment

```xml
<ldap-realm name="exampleLDAPRealm" dir-context="exampleDirContext"> 
    <identity-mapping rdn-identifier="uid" search-base-dn="ou=Users,dc=wildfly,dc=org"> 
        <attribute-mapping> 
            <attribute from="cn" to="Roles" filter="(&amp;(objectClass=groupOfNames)(member={1}))" filter-base-dn="ou=Roles,dc=wildfly,dc=org"/> 
        </attribute-mapping>
        <user-password-mapper from="userPassword"/> 
    </identity-mapping>
</ldap-realm>
```


## JDBC realm
1. Deploy the database driver for the database using the management CLI.

deploy PATH_TO_JDBC_DRIVER/postgresql-42.2.9.jar

2. Configure the database as the data source.
data-source add --name=examplePostgresDS --jndi-name=java:jboss/examplePostgresDS --driver-name=postgresql-42.2.9.jar  --connection-url=jdbc:postgresql://localhost:5432/postgresdb --user-name=postgres --password=postgres

3. Create a jdbc-realm in Elytron

/subsystem=elytron/jdbc-realm=exampleSecurityRealm:add(principal-query=[{sql="SELECT password,roles FROM example_jboss_eap_users WHERE username=?",data-source=examplePostgresDS,clear-password-mapper={password-index=1},attribute-mapping=[{index=2,to=Roles}]}])
{"outcome" => "success"}

4. Create a security domain that references the jdbc-realm.

/subsystem=elytron/security-domain=exampleSecurityDomain:add(default-realm=exampleSecurityRealm,permission-mapper=default-permission-mapper,realms=[{realm=exampleSecurityRealm}])
{"outcome" => "success"}

5. To verify that Elytron can load data from the database, use the following command:

/subsystem=elytron/security-domain=exampleSecurityDomain:read-identity(name=user1)
{
    "outcome" => "success",
    "result" => {
        "name" => "user1",
        "attributes" => {"Roles" => ["Admin"]},
        "roles" => ["Admin"]
    }
}

