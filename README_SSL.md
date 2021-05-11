Features enabled by the SSL/TLS playbook
1. Encrypt communication with elasticsearch.
See the oficial documentation on details of the implementation: 
[Encrypt traffic between Kibana and Elasticsearch](https://www.elastic.co/guide/en/kibana/current/configuring-tls.html#configuring-tls-kib-es)
and [Configure Kibana to use the client certificate and private key.](https://www.elastic.co/guide/en/kibana/current/elasticsearch-mutual-tls.html)

To enable this module simple set the variable
```yaml
es_enable_http_ssl: true
```

       
a. If your certificate and private key are contained in a PKCS#12 (*.p12 or *.pfx) file then use 
```yaml
es_ssl_keystore: "files/certs/my-keystore.p12"
es_ssl_keystore_password: elastic
```
b. Otherwise, if your certificate and private key are in PEM format:
```yaml
es_ssl_cert: "files/certs/client/client.crt"
es_ssl_key: "files/certs/client/client.key"
es_ssl_key_passphrase: elastic
```         


2. Kibana Server SSL for Encrypt traffic between the browser and Kibana
                          as described in [Encrypt Communications in Kibana](https://www.elastic.co/guide/en/kibana/current/configuring-tls.html)

Set the variable:
```yaml
enable_server_ssl: true
```                          
a. If your Kibana server certificate and private key are contained in a PKCS#12  (*.p12 or *.pfx) file then use

```yaml
server_ssl_keystore: "files/certs/my-keystore.p12"
server_ssl_keystore_password: elastic
```

b. If your server certificate and private key are in PEM format:
```yaml
server_ssl_cert: "files/certs/server/server.crt"
server_ssl_key: "files/certs/server/server.key"
server_ssl_key_passphrase: elastic
```                    
3. Implement [Secure Settings](https://www.elastic.co/guide/en/kibana/current/secure-settings.html)

Stores the Kibana username/password in he Kibana keystore instead of the kibana.conf file.

`secure_settings: true`
4. Implement OIDC kibana configuration as documented here
   [Configuring Kibana](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/oidc-kibana.html) and  
   [Set up OpenID Connect with Azure, Google, or Okta](https://www.elastic.co/guide/en/cloud/7.10/ec-securing-clusters-oidc-op.html)
   [Secure your clusters with OpenID Connect]( https://www.elastic.co/guide/en/cloud-heroku/7.10/ech-secure-clusters-oidc.html)
    
This is controlled via `es_enable_oidc: true`
And will add the following settings to `kibana.yml`
```yaml
xpack.security.authc.providers:
  oidc.oidc1:
    order: 0
    realm: oidc1
    description: "Log in with my OpenID Connect"
  basic.basic1:
    order: 1
```   

If you are using a Kibana instance of version 7.6 or earlier change the settings in your `kibana.yml` to:
```yaml
xpack.security.authc.providers: [oidc]
xpack.security.authc.oidc.realm: "oidc1" 
server.xsrf.whitelist: [/api/security/v1/oidc]
```
## Here is a sample Playbook with all the variables

```yaml
  - name: Simple Kibana Playbook with SSL enabled 
    hosts: kibana-node
    roles:
    - role: fedelemantuano.kibana
      es_version: 7.11.2
      kibana_api_host: "{{ ansible_default_ipv4.address }}"
      #Secure communication with Elasticsearch
      es_enable_http_ssl: true
      es_ssl_verification_mode: "certificate"
      #Use se the PKCS12 .p12 or pfx keystore file.
      es_ssl_keystore: "files/certs/my-keystore.p12""
      es_ssl_keystore_password: elastic
      #Alternatively use the certificate/key 
      #es_ssl_cert: "files/certs/client/client.crt"
      #es_ssl_key: "files/certs/client/client.key"
      #es_ssl_key_passphrase: elastic
      enable_server_ssl: true
      server_ssl_keystore: "files/certs/my-keystore.p12"
      server_ssl_keystore_password: elastic
      #server_ssl_cert: "files/certs/server/server.crt"
      #server_ssl_key: "files/certs/server/server.key"
      #server_ssl_key_passphrase: elastic
      #Store kibana settings in the keystore
      secure_settings: true
      es_user: kibana_system
      es_pass: changeme
      es_enable_oidc: false
      kibana_config:
          server.name: "{{ inventory_hostname }}"
          server.port: 5601
          server.host: "{{ ansible_default_ipv4.address }}"
          elasticsearch.hosts: "https://{{ ansible_default_ipv4.address }}:9200"
          xpack.security.audit.enabled: true
          #Add this when deploying behing AWS ALB Target Group
          #server.basePath: "/kibana"
          #server.rewriteBasePath: true


```



### Debugging tips

Check what is stored in the Kibana keystore. 
```shell script
[root@kibana-host]# /usr/share/kibana/bin/kibana-keystore list  --allow-root
elasticsearch.ssl.keystore.password
server.ssl.keystore.password
elasticsearch.username
elasticsearch.password
[root@kibana-host]#
``` 

### Removing Kibana 

Use this playbook for removing kibana from RPM based system.
```yaml
- hosts:  kibana-node
  tasks:
    - yum:
        name: kibana
        state: absent
      become: true
    - file:
        path: "{{item}}"
        state: absent
      loop:
        - /etc/kibana
        - /usr/share/kibana
        - /var/lib/kibana
      become: true

```
