# Übersicht
![apim-mi_flow.jpg](/.attachments/apim-mi_flow-03f7916d-417e-4af2-9527-371c03ae7747.jpg)

## 1. JWT mit Postman holen
- Authorization Type: OAuth 2.0

|Feldname|Wert  |
|--|--|
| Grant type | Authorization Code|
|  Auth URL| https://login.microsoftonline.com/`<your-tenant-id>`/oauth2/v2.0/authorize |
| Access Token URL | https://login.microsoftonline.com/`<your-tenant-id>`/oauth2/v2.0/token |
| Client ID | <**Client** App Registration **Application (client) ID**> |
| Client Secret | <**Client** App Registration **postmanSecret**> |
|Scope|api://<**API** App Registration **Application (client) ID**>/user_impersonation|

## 2. APIM mit JWT aufrufen
![image.png](/.attachments/image-fc2808c4-d81b-4612-801c-2836b7a343e4.png)
## 3. APIM validiert JWT gegen Client App Registration
```
<validate-azure-ad-token tenant-id="<your-tenant-id>">
    <client-application-ids>
        <application-id><Client App Registration Application (client) ID></application-id>
    </client-application-ids>
    <required-claims>
       <claim name="scp" match="any" separator=" ">
           <value>user_impersonation</value>
        </claim>
    </required-claims>
</validate-azure-ad-token>
```

## 4. APIM authentifiziert sich selbst mit Managed Identity gegen App Registration von API
```
<authentication-managed-identity resource="<API App Registration Application (client) ID>" />
```

## 5. APIM ruft API authentifiziert auf (mit function code key)

# Implementierung
## 0. Ausgangslage

![apim-mi.jpg](/.attachments/apim-mi-1489aee5-8d59-4cba-9148-75893261f83f.jpg)
### APIM
![image.png](/.attachments/image-de3450bd-3e67-4076-bbb0-7e84294e75ef.png)

## 1. Authentication in Azure Function aktivieren
[Doku](https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-provider-aad?tabs=workforce-configuration#daemon-client-application-service-to-service-calls)
1. Add an identity provider ![image.png](/.attachments/image-11e9b545-390d-42e7-956b-079fd11d3d14.png)
2. Create new app registration

    |Feldname|Wert  |
    |--|--|
    |Allowed client applications  | APIM MI Application ID |


    ![image.png](/.attachments/image-7b976331-2ce3-4297-8afc-0858d9a97ee9.png)
    ![image.png](/.attachments/image-1d6aa4cf-a0d1-4ffe-a1df-63f44cd32639.png)
    ![image.png](/.attachments/image-66685b6d-3ef3-41ce-8bd4-748851672b20.png)

3. Restart function

    ![image.png](/.attachments/image-c7e74bfb-ae6b-4fba-9a9d-c1c1c5aab904.png)

4. Check function access restriction is active
![image.png](/.attachments/image-f90548f6-fec5-48f6-b97e-60219615d312.png)
5. Add Managed Identity authentication to APIM policy

    |Feldname|Wert  |
    |--|--|
    | resource  | mytestbackendfnct Application (client) ID |

    ![image.png](/.attachments/image-edf4f6aa-947f-4e5e-868d-6da302fdfea7.png)
6. Verify successful authentication and access
    ![image.png](/.attachments/image-b2901000-59f3-4e61-8884-dc1ff737af6c.png)
7. Add app registration for client in Entra ID
    
    ![image.png](/.attachments/image-dbeb447e-9901-46e7-b032-0bc792337906.png)
    ![image.png](/.attachments/image-8874dde6-a814-4a9a-a4b6-68fd3c2ddb25.png)
8. Add API permission to app registration
    ![image.png](/.attachments/image-bf4335b7-1d0c-4356-b57c-80de2cfa3290.png)
    ![image.png](/.attachments/image-f555b2b3-c353-4727-b35a-1ef20dff9f23.png)
9. Add authentication for Postman
    ![image.png](/.attachments/image-6b9a1a62-80ff-4791-a54a-6fb3919cc61d.png)
10. create client secret for Postman
    ![image.png](/.attachments/image-0fa97898-349f-4e2b-aca5-32aaf54d9ff3.png)
11. Implement Pre-auth mit JWT validation in APIM Policy
    **Reihenfolge der Policies ist wichtig!!** `authentication-managed-identity` muss **nach** `validate-azure-ad-token` kommen
    |Feldname|Wert  |
    |--|--|
    | application-id| myClient4mytestbackendfnct Application (client) ID |

    ![image.png](/.attachments/image-5cc53ed5-f504-4ad6-a5a6-df35d66fc093.png)
12. Check access restriction
    ![image.png](/.attachments/image-193f033d-35fc-4acf-8438-718a336ca2e2.png)
13. Acquire Token in Postman
    13.1 Get Scope
    ![image.png](/.attachments/image-4ddc75f2-efec-467f-8775-704ea8348ae4.png)
    |Feldname|Wert  |
    |--|--|
    | Grant type | Authorization Code|
    |  Auth URL| https://login.microsoftonline.com/<your-tenant-id>/oauth2/v2.0/authorize |
    | Access Token URL | https://login.microsoftonline.com/<your-tenant-id>/oauth2/v2.0/token |
    | Client ID | myClient4mytestbackendfnct  |
    | Client Secret | `<your-client-secret-value>` |
    |Scope|api://<mytestbackendfnct-application-id>/user_impersonation|

    ![image.png](/.attachments/image-d590c5f2-150d-4741-80d2-1486af7db99f.png)
    ![image.png](/.attachments/image-2f8147ac-fcc3-4d09-9e16-eff93acf97fd.png)
    ![image.png](/.attachments/image-6a12cb1a-465a-4a1c-af84-4272a191bc0a.png)
    ![image.png](/.attachments/image-6e848eaf-4b2c-4637-9e13-e1b9c2b5c1fa.png)
    ![image.png](/.attachments/image-ff90ab08-5732-4eed-a326-363d9a3f9671.png)

14. Use Token in APIM
    ![image.png](/.attachments/image-8d8f5ba1-21f3-4263-b6f6-65554197fb84.png)

