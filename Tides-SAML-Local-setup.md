# Prerequisite
- elasticsearch with `x_pack security` enabled
- elasticsearch with `trial licence` or above
- `loginStrategy` in tides.yml should be 'saml'
# Download idp metadata file
- https://samltest.id/saml/idp
- Save metadata file as `idp-metadata.xml`to elasticsearch's `config/` directory 
# Elasticsearch setup
###  - Enable the token service
   - The Elasticsearch SAML implementation makes use of the Elasticsearch Token Service. This service is automatically enabled if you configure TLS on the HTTP interface, and can be explicitly configured by including the following in your `elasticsearch.yml` file:
    - `xpack.security.authc.token.enabled: true`
    

###  - Create a SAML realm
- Create a realm by adding the following to your `elasticsearch.yml` configuration file.
``` 
xpack.security.authc.realms.saml.saml1:
    order: 2
      idp.metadata.path: idp-metadata.xml
      idp.entity_id: "https://samltest.id/saml/idp"
      sp.entity_id: "http://0.0.0.0:5601"
      sp.acs: "http://0.0.0.0:5601/api/guard/saml"
      sp.logout: "http://0.0.0.0:5601/app/logout"
      attributes.principal: "urn:oid:0.9.2342.19200300.100.1.1"
      attributes.groups: "urn:oid:1.3.6.1.4.1.5923.1.5.1."
 ```
# Generate SP metadata
- Run `bin/elasticsearch-saml-metadata` command in your Elasticsearch directory.
- Above command will generate SP metadata file in your Elasticsearch directory

# Upload SP metadata to idp
- Use this URL to upload SP metadata generated in previous step : https://samltest.id/upload.php

# Configuring role mappings
- When a user authenticates using SAML, they are identified to the Elastic Stack, but this does not automatically grant them access to perform any actions or access any data.
- Your SAML users cannot do anything until they are assigned roles. This can be done through `add role mapping API`
- You can create role mapping from  security with `basic loginStrategy enabled` (which is enabled by default), OR with tools like Sense or CURL
```
PUT /_security/role_mapping/saml-admin
{
  "roles": [ "superuser" ],
  "enabled": true,
  "rules": {
    "field": { "realm.name": "saml1" }
  }
}
```
- Above example of a simple role mapping that grants the `superuser` role to any user who authenticates against the saml1 realm

# TIDES Setup 
- `loginStrategy` in tides.yml should be 'saml'
