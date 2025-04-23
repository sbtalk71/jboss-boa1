## TLS For JBoss EAp 8.0 with elytron

1. Start eleytron in intercative mode. COnnect to Jboss CLi and use the following command:
```sh
security enable-ssl-management --interactive
```
2. This will configure:

	- key-store
	- key-manager
	- server-ssl-context

 The server-ssl-context is then applied to http-interface.

 Elytron names each resource as resource-type-UUID. For example, key-store-9e35a3be-62bb-4fff-afc2-2d8d141b82bc.

3. Create a client side trust store
```sh
keytool -importcert -keystore client.truststore.pkcs12 -storepass secret -alias localhost -trustcacerts -file exampleKeystore.pem
```
4. Define the client-side SSL context in a file, for example example-security.xml.
```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>
    <authentication-client xmlns="urn:elytron:client:1.2">
        <key-stores>
            <key-store name="clientStore" type="PKCS12" >
                <file name="JBOSS_HOME/standalone/configuration/client.truststore.pkcs12"/>
                <key-store-clear-password password="secret" />
            </key-store>
        </key-stores>
        <ssl-contexts>
            <ssl-context name="client-SSL-context">
                <trust-store key-store-name="clientStore" />
            </ssl-context>
        </ssl-contexts>
        <ssl-context-rules>
            <rule use-ssl-context="client-SSL-context" />
        </ssl-context-rules>
    </authentication-client>
</configuration>
```

5. Connect to the server and issue a command.
```sh
EAP_HOME/bin/jboss-cli.sh -c --controller=remote+https://127.0.0.1:9993 -Dwildfly.config.url=<path_to_the_configuration_file>/example-security.xml :whoami
```
6. Inspect the certificate and verify that the fingerprints shown in your browser match the fingerprints of the certificate in your keystore. You can view the certificate you generated with the following command:

```sh
/subsystem=elytron/key-store=key-store-a18ba30e-6a26-4ed6-87c5-feb7f3e4dff1:read-alias(alias="localhost")
```

## SSL Using subsystem command
1. Add a keystore
```sh
/subsystem=elytron/key-store=exampleKeyStore:add(path=exampleserver.keystore.pkcs12, relative-to=jboss.server.config.dir,credential-reference={clear-text=secret},type=PKCS12)
```

2. If the keystore doesn’t contain any certificates, or you used the step above to create the keystore, you must generate a certificate and store the certificate in a file.

Generate a key pair in the keystore.
```sh
/subsystem=elytron/key-store=exampleKeyStore:generate-key-pair(alias=localhost,algorithm=RSA,key-size=2048,validity=365,credential-reference={clear-text=secret},distinguished-name="CN=localhost")
```

3. Store the certificate in a file.
```sh
/subsystem=elytron/key-store=exampleKeyStore:store()
```

4. Configure a key-manager referencing the key-store.
```sh
/subsystem=elytron/key-manager=exampleKeyManager:add(key-store=exampleKeyStore,credential-reference={clear-text=secret})
```

4. Configure a server-ssl-context referencing the key-manager.
```sh
/subsystem=elytron/server-ssl-context=examplehttpsSSC:add(key-manager=exampleKeyManager, protocols=["TLSv1.2"])
```
5. Update the management interfaces to use the configured server-ssl-context.
```sh
/core-service=management/management-interface=http-interface:write-attribute(name=ssl-context, value=examplehttpsSSC)
/core-service=management/management-interface=http-interface:write-attribute(name=secure-socket-binding, value=management-https)
```
