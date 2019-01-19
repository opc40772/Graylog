# Graylog Server.

Graylog version: 2.5.1

Elasticsearch version: 5.6.8 and 6.5.4

Create indice for zimbra. In System / Indices. The index prefix must be zimbra as the image show.
This is important for the proper functioning of the streams.
![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Indices_and_Index_Sets_-_2018-03-07_13.png)

Upload the file in folder Content Pack.

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Content_packs_-_2018-03-08_10.32.39.png)

This content pack have grok patterns, beats input for zimbra and stream for zimbra with it's rules

The beats inputs port:5045

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Inputs_-_2018-03-05_14.png)

Edit zimbra the stream and select the index previusly created

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Streams_-_2018-03-09_11.28.49.png)


# Zimbra Server

You must install filebeat 5.6.x for elasticsearch v5 or filebeat 6.5.x for elasticsearch 6, for any other version of elasticsearch .See [Matrix compatibility](https://www.elastic.co/support/matrix#matrix_compatibility)

In Filebeat 5.6.x use this config...

We will only modify the sessions of Filebeat prospectors and Logstash output.

    #=================== Filebeat prospectors ======================
    filebeat.prospectors:
     
    # Each – is a prospector. Most options can be set at the prospector level, so
    # you can use different prospectors for various configurations.
    # Below are the prospector specific configurations.
 
    – input_type: log
    document_type: postfix
    paths:
    – /var/log/mail.log
    – input_type: log
    document_type: zimbra_audit
    paths:
    – /opt/zimbra/log/audit.log
    – input_type: log
    document_type: zimbra_mailbox
    paths:
    – /opt/zimbra/log/zmmailboxd.out
    – input_type: log
    document_type: nginx
    paths:
    – /opt/zimbra/log/nginx.access.log
 
    #———————- Logstash output —————————
    output.logstash:
    # The Logstash hosts
    hosts: ["graylog.dominio.com:5045"]
 
    # Optional SSL. By default is off.
    # List of root certificates for HTTPS server verifications
    bulk_max_size: 2048
    #ssl.certificate_authorities: ["/etc/filebeat/graylog.crt"]
    template.name: "filebeat"
    template.path: "filebeat.template.json"
    template.overwrite: false
    # Certificate for SSL client authentication
    #ssl.certificate: "/etc/pki/client/cert.pem"
 
    # Client Certificate Key
    #ssl.key: "/etc/pki/client/cert.key"

In Filebeat 6.5.x use this config...

We will only modify the sessions of Filebeat inputs and Logstash output.

    #=========================== Filebeat inputs =============================

    filebeat.inputs:

    - type: log
    paths:
    - /var/log/mail.log
    fields_under_root: true
    fields:
        type: postfix
    - type: log
    fields_under_root: true
    fields:
        type: zimbra_audit
    paths:
        - /opt/zimbra/log/audit.log
    - type: log
    fields_under_root: true
    fields:
        type: zimbra_mailbox
    paths:
        - /opt/zimbra/log/mailbox.log
    - type: log
    fields_under_root: true
    fields:
        type: nginx
    paths:
        - /opt/zimbra/log/nginx.access.log
    
    #———————- Logstash output —————————
    output.logstash:
    # The Logstash hosts
    hosts: ["graylog.dominio.com:5045"]
 
    # Optional SSL. By default is off.
    # List of root certificates for HTTPS server verifications
    bulk_max_size: 2048
    #ssl.certificate_authorities: ["/etc/filebeat/graylog.crt"]
    template.name: "filebeat"
    template.path: "filebeat.template.json"
    template.overwrite: false
    # Certificate for SSL client authentication
    #ssl.certificate: "/etc/pki/client/cert.pem"
 
    # Client Certificate Key
    #ssl.key: "/etc/pki/client/cert.key"

The host in Logstash output section is graylog server.

