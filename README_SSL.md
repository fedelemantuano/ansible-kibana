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

```yaml
  - name: Simple Example with SSL 
    hosts: kibana-node
    roles:
    - role: fedelemantuano.kibana
      es_version: 7.6.2
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
      es_user: kibana
      es_pass: changeme
      kibana_config:
          server.name: "{{ inventory_hostname }}"
          server.port: 5601
          server.host: "{{ ansible_default_ipv4.address }}"
          elasticsearch.hosts: "https://{{ ansible_default_ipv4.address }}:9200"
          xpack.security.audit.enabled: true


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