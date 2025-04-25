# Steps to Create a Database Realm in JBoss EAP 8.0

This guide outlines the steps to create a database realm in JBoss EAP 8.0 using Elytron.

## Prerequisites

- JBoss EAP 8.0 installed and configured.
- A relational database (MySQL, PostgreSQL, etc.) with a users table.
- JDBC driver module added to JBoss.

## Step-by-Step Guide

### 1. Add JDBC Driver Module

```bash
$JBOSS_HOME/bin/jboss-cli.sh --connect
module add --name=com.mysql --resources=/path/to/mysql-connector-java-x.x.x.jar --dependencies=javax.api,javax.transaction.api
```

### 2. Add a Datasource

```bash
/subsystem=datasources/data-source=ExampleDS:add(
    jndi-name=java:/jdbc/exampleDS,
    driver-name=mysql,
    connection-url=jdbc:mysql://localhost:3306/exampledb,
    user-name=dbuser,
    password=dbpassword
)
```

### 3. Define a Database Realm

```bash
/subsystem=elytron/jdbc-realm=MyDatabaseRealm:add(
    principal-query=[{
        sql="SELECT password FROM users WHERE username = ?",
        data-source=ExampleDS,
        clear-password-mapper={password-index=1}
    }]
)
```

### 4. Create a Security Domain

```bash
/subsystem=elytron/security-domain=MySecurityDomain:add(
    default-realm=MyDatabaseRealm,
    realms=[{realm=MyDatabaseRealm, role-decoder=groups-to-roles}],
    permission-mapper=default-permission-mapper
)
```

### 5. Configure an Application Security Domain

```bash
/subsystem=undertow/application-security-domain=MyAppDomain:add(
    security-domain=MySecurityDomain,
    http-authentication-factory=application-http-authentication
)
```

### 6. Reference the Security Domain in Your Deployment

In your `web.xml`:

```xml
<login-config>
    <auth-method>BASIC</auth-method>
    <realm-name>MyAppDomain</realm-name>
</login-config>
```

And in `jboss-web.xml`:

```xml
<jboss-web>
    <security-domain>MyAppDomain</security-domain>
</jboss-web>
```

## Notes

- Adjust the SQL query based on your schema.
- Add additional queries for roles if needed.
- Secure your database credentials properly.

