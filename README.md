# jboss-boa1

# Post assessment and Feedback
 
Post assessment: https://forms.office.com/r/70RW6xZgtH

Feedback: https://forms.office.com/r/eP0fT8U6r7

# JDK-11 Download Link

```sh
https://cdn.azul.com/zulu/bin/zulu11.80.21-ca-jdk11.0.27-win_x64.zip

```

# RBAC Enablement XML Fragment

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
