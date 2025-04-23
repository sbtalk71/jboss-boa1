# jboss-boa1

```xml
<access-control provider="rbac">
            <role-mapping>
                <role name="SuperUser">
                    <include>
                        <user name="$local"/>
                    </include>
                </role>
            </role-mapping>
     <role-mapping>
                <role name="Administrator">
                    <include>
                        <group name="admin"/>
                    </include>
                </role>
            </role-mapping>
<role-mapping>
     <role name="Monitor">
                    <include>
                        <group name="monitoring"/>
                    </include>
                </role>
            </role-mapping>
        </access-control>
```
