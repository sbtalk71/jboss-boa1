# Elytron security with JBoss EAP 8.0 for Datasource
---
username: root
password : root
## Architecture: Datasource[uses]-> authentication-context[uses]->authentication-configuration[uses]->credential-store

### Steps:
1. **Configure credential store either with `elytron-tool` or `jboss-cli script`**
	- elytron-tool
	```sh
	elytron-tool.bat credential-store --create --location=myCredStore.cs --password=MyStorePassword --add root --secret=root
	```
	- cli script
	```sh
	/subsystem=elytron/credential-store=myCredStore:add(path=myCredStore.cs,relative-to=jboss.server.config.dir,credential-reference={clear-text=MyStorePassword},create=true)
	```
	- The above will add the given xml fragment
	```xml
	<credential-stores>
      <credential-store name="myCredStore" relative-to="jboss.server.config.dir" path="myCredStore.cs" create="true">
         <credential-reference clear-text="MyStorePassword"/>
       </credential-store>
    </credential-stores>
	```
2. **Configure Authentication-configuration**
	```sh
	/subsystem=elytron/authentication-configuration=db-auth-config:add(credential-reference={store=myCredStore,alias=root},authentication-name=root)
	```
3. **Configure authentication-context**
	```sh
	/subsystem=elytron/authentication-context=db-auth-context:add(match-rules=[{match-host="*",authentication-configuration=db-auth-config}])
	```
4. **Create the DataSource and provide the ref to the authentication-context**

	```sh
	/subsystem=datasources/data-source=mysqlDS:write-attribute(name=elytron-enabled,value=true)
	
	/subsystem=datasources/data-source=mysqlDS:write-attribute(name=authentication-context,value=db-auth-context)
	```
5. **Finally We should get the XML Entry in datasource Element as given :**
	```xml
	<security>
           <elytron-enabled>true</elytron-enabled>
           <authentication-context>db-auth-context</authentication-context>
    </security>
	```

 # Alternative Approach
 
 1. **Configure the credential store as given above *Step1***
 2. **Use the following xml with <datasource> element**
 ```xml
			<security>
                        <user-name>root</user-name>
                        <credential-reference name="myCredStore" alias="root"/>
			</security>
 ```