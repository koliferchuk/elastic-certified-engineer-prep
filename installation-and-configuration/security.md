## Installation and Configuration

#### Enable xPack security on the cluster

Update configuration on `node1`: 

```
xpack.security.enabled: true
```

Generate a certificate: 

```
bin/elasticsearch-certutil ca
```

Enable TSL:
This requirement applies to clusters with more than one node and to clusters with a single node that listens on an external interface. Single-node clusters that use a loopback interface do not have this requirement.

```
pack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.keystore.path: elastic-stack-ca.p12
xpack.security.transport.ssl.truststore.path: elastic-stack-ca.p12
```

#### Set the password of the `elastic` and `kibana` built-in users, by using the pattern "{{username}}-password" (e.g., "elastic-password")

```
bin/elasticsearch-setup-passwords interactive
```

#### Login to Kibana using the `elastic` user credentials

Update `bin/kibana.yml` & restart Kibana: 

```
elasticsearch.password: elastic-password
```

#### Create the index `hamlet` and add some documents by running the following _bulk command

```
PUT hamlet/_doc/_bulk
{"index":{"_index":"hamlet","_id":0}}
{"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet","_id":1}}
{"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet","_id":2}}
{"line_number":"3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet","_id":3}}
{"line_number":"4","speaker":"FRANCISCO","text_entry":"Bernardo?"}
{"index":{"_index":"hamlet","_id":4}}
{"line_number":"5","speaker":"BERNARDO","text_entry":"He."}
{"index":{"_index":"hamlet","_id":5}}
{"line_number":"6","speaker":"FRANCISCO","text_entry":"You come most carefully upon your hour."}
{"index":{"_index":"hamlet","_id":6}}
{"line_number":"7","speaker":"BERNARDO","text_entry":"Tis now struck twelve; get thee to bed, Francisco."}
{"index":{"_index":"hamlet","_id":7}}
{"line_number":"8","speaker":"FRANCISCO","text_entry":"For this relief much thanks: tis bitter cold,"}
{"index":{"_index":"hamlet","_id":8}}
{"line_number":"9","speaker":"FRANCISCO","text_entry":"And I am sick at heart."}
{"index":{"_index":"hamlet","_id":9}}
{"line_number":"10","speaker":"BERNARDO","text_entry":"Have you had quiet guard?"}
```

#### Create the security role `francisco_role` in the native realm, so that the role (i) has "monitor" privileges on the cluster, (ii) has all privileges on the `hamlet` index

Use Security API: 
```
POST _security/role/francisco_role
{
  "indices" : [
    {
      "names" : [ "hamlet" ],
      "privileges" : [ "all" ]
    }
  ],
  "cluster": ["monitor"]
}
```