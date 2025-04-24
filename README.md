# jboss-boa1
``sh
https://cdn.azul.com/zulu/bin/zulu17.58.21-ca-jdk17.0.15-win_x64.msi?_gl=1*1up5jya*_gcl_au*MTM5MDY5MzY4OS4xNzQ1NDc5Mzcw*_ga*NjE2Mzg4MTY1LjE3NDU0NzkzNzA.*_ga_42DEGWGYD5*MTc0NTQ3OTM2OS4xLjEuMTc0NTQ3OTQzMC41OS4wLjE4OTc5OTk2NDQ.
``
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
