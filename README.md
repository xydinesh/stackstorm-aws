# AWS Integration Pack

The StackStorm AWS integration pack supplies action integration for EC2 and Route53.

## Configuration

You will need to add a set of AWS credentials, and default zone to the config.yaml file:

```yaml
---
setup:
  region: ""
  aws_access_key_id: ""
  aws_secret_access_key: ""
st2_user_data: ""
 ```

You can generate the access key and secret access key by following these directions:

http://docs.aws.amazon.com/IAM/latest/UserGuide/ManagingCredentials.html#Using_CreateAccessKey

If you would like to use the IAM role assigned to the instance stackstorm is running set the key and secret to null and set the region.
```yaml
---
setup:
  region: "us-east-1"
  aws_access_key_id: null
  aws_secret_access_key: null
st2_user_data: ""
 ```

* ``service_notifications_sensor.host`` - Listen host for the HTTP interface.
* ``service_notifications_sensor.port`` - Listen port for the HTTP interface.
* ``service_notifications_sensor.path`` - Path where the events need to be sent.

And `aws.example.file` is an example of this configuration. You can use actions and sensors by copying this file to `config.yaml` and editing the parameters in this file.

**Note** : When modifying the configuration in `/opt/stackstorm/configs/` please
           remember to tell StackStorm to load these new values by running
           `st2ctl reload --register-configs`

## st2_user_data

Optionally, you can set the user_data to set a default file to be used during new instance creation.  Put your user_data file somewhere accessible by the StackStorm user, and use the st2_user_data config option to set it.

```yaml
st2_user_data: "/full/path/to/file"
 ```

This file/script will be used for all invocations of the ec2_run_instances action

## Sensors

### ServiceNotificationsSensor

This sensor exposes a HTTP interface and listens for service event
notifications which are delivered via AWS SNS service using HTTP(s) endpoint.

Currently it supports event notifications generated by the S3 service -
http://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html

Keep in mind that this sensor doesn't implement any kind of authentication.
This means that if the sensor is not behind a proxy which implements
authentication and is accessible to the outside world, you need to configure
``path`` setting to include a random secret which is only known to the
AWS SNS service.

#### aws.service_notification

This trigger is emitted for every service event notification.

Example trigger payload:

```json
{
    "source": "aws:s3",
    "region": "us-west-2",
    "name": "ObjectCreated:Put",
    "timestamp": 123456789,
    "response_elements": {
        "x-amz-id-2": "blahid//4U+rk=",
        "x-amz-request-id": "5FF5BB6EDE3631F8"
    },
    "request_parameters": {
        "sourceIPAddress": "127.0.0.1"
    },
    "payload": {
        "configurationId": "snsnotificationforput",
        "object": {
            "eTag": "5dfd7f29bce6d94dc5c73553f269659b",
            "key": "myfile.tar.gz",
            "size": 178428
        },
        "bucket": {
            "ownerIdentity": {
                "principalId": "BLAHBLAH"
            },
            "name": "testbucket33333",
            "arn": "arn:aws:s3:::testbucket33333"
        },
        "s3SchemaVersion": "1.0"
    }
}
```

``source``, ``region``, ``name`` and ``timestamp`` attributes are included with
all the events, but the format and values inside the ``payload`` attribute
different across different services and event types.

For a list of supported S3 event types see
http://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html#supported-notification-event-types

### AWS SQS sensor

This is generic *SQS* Sensor using boto3 api to fetch messages from *SQS* queue.
After receiving a message it's content is passed as payload to a trigger 'aws.sqs_new_message'

This sensor can be configured either by using config.yaml within a pack or by creating
following values in datastore:

- aws.input_queues (list queues as comma separated string: first_queue,second_queue)
- aws.aws_access_key_id
- aws.aws_secret_access_key
- aws.region
- aws.max_number_of_messages (must be between 1 - 10)

For configuration in ``config.yaml`` with config like this

```yaml
    setup:
      aws_access_key_id:
      aws_access_key_id:
      region:
    sqs_sensor:
      input_queues:
        - first_queue
        - second_queue
    sqs_other:
      max_number_of_messages: 1
```

If any value exist in datastore it will be taken instead of any value in config.yaml

#### aws.sqs\_new\_message

This trigger is emitted when a single message is received from a queue.

```json
{
  "queue": "first_sqs_queue",
  "body": "example message body"
}
```

## Actions

### Route53 Actions

* r53\_build\_base\_http\_request
* r53\_change\_rrsets
* r53\_close
* r53\_create\_health\_check
* r53\_create\_hosted\_zone
* r53\_create\_zone
* r53\_delete\_health\_check
* r53\_delete\_hosted\_zone
* r53\_get\_all\_hosted\_zones
* r53\_get\_all\_rrsets
* r53\_get\_change
* r53\_get\_hosted\_zone
* r53\_get\_hosted\_zone\_by\_name
* r53\_get\_http\_connection
* r53\_get\_list\_health\_checks
* r53\_get\_path
* r53\_get\_proxy\_auth\_header
* r53\_get\_proxy\_url\_with\_auth
* r53\_get\_zone
* r53\_get\_zones
* r53\_handle\_proxy
* r53\_make\_request
* r53\_new\_http\_connection
* r53\_prefix\_proxy\_to\_path
* r53\_proxy\_ssl
* r53\_put\_http\_connection
* r53\_server\_name
* r53\_set\_host\_header
* r53\_set\_request\_hook
* r53\_skip\_proxy
* r53\_zone\_add\_a
* r53\_zone\_add\_cname
* r53\_zone\_add\_mx
* r53\_zone\_add\_record
* r53\_zone\_delete
* r53\_zone\_delete\_a
* r53\_zone\_delete\_cname
* r53\_zone\_delete\_mx
* r53\_zone\_delete\_record
* r53\_zone\_find\_records
* r53\_zone\_get\_a
* r53\_zone\_get\_cname
* r53\_zone\_get\_mx
* r53\_zone\_get\_nameservers
* r53\_zone\_get\_records
* r53\_zone\_update\_a
* r53\_zone\_update\_cname
* r53\_zone\_update\_mx
* r53\_zone\_update\_record

### EC2 Actions

* ec2\_allocate\_address
* ec2\_assign\_private\_ip\_addresses
* ec2\_associate\_address
* ec2\_associate\_address\_object
* ec2\_attach\_network\_interface
* ec2\_attach\_volume
* ec2\_authorize\_security\_group
* ec2\_authorize\_security\_group\_deprecated
* ec2\_authorize\_security\_group\_egress
* ec2\_build\_base\_http\_request
* ec2\_build\_complex\_list\_params
* ec2\_build\_configurations\_param\_list
* ec2\_build\_filter\_params
* ec2\_build\_list\_params
* ec2\_build\_tag\_param\_list
* ec2\_bundle\_instance
* ec2\_cancel\_bundle\_task
* ec2\_cancel\_reserved\_instances\_listing
* ec2\_cancel\_spot\_instance\_requests
* ec2\_close
* ec2\_confirm\_product\_instance
* ec2\_copy\_image
* ec2\_copy\_snapshot
* ec2\_create\_image
* ec2\_create\_key\_pair
* ec2\_create\_network\_interface
* ec2\_create\_placement\_group
* ec2\_create\_reserved\_instances\_listing
* ec2\_create\_security\_group
* ec2\_create\_snapshot
* ec2\_create\_spot\_datafeed\_subscription
* ec2\_create\_tags
* ec2\_create\_volume
* ec2\_delete\_key\_pair
* ec2\_delete\_network\_interface
* ec2\_delete\_placement\_group
* ec2\_delete\_security\_group
* ec2\_delete\_snapshot
* ec2\_delete\_spot\_datafeed\_subscription
* ec2\_delete\_tags
* ec2\_delete\_volume
* ec2\_deregister\_image
* ec2\_describe\_account\_attributes
* ec2\_describe\_reserved\_instances\_modifications
* ec2\_describe\_vpc\_attribute
* ec2\_detach\_network\_interface
* ec2\_detach\_volume
* ec2\_disassociate\_address
* ec2\_enable\_volume\_io
* ec2\_get\_all\_addresses
* ec2\_get\_all\_bundle\_tasks
* ec2\_get\_all\_images
* ec2\_get\_all\_instance\_status
* ec2\_get\_all\_instance\_types
* ec2\_get\_all\_instances
* ec2\_get\_all\_kernels
* ec2\_get\_all\_key\_pairs
* ec2\_get\_all\_network\_interfaces
* ec2\_get\_all\_placement\_groups
* ec2\_get\_all\_ramdisks
* ec2\_get\_all\_regions
* ec2\_get\_all\_reservations
* ec2\_get\_all\_reserved\_instances
* ec2\_get\_all\_reserved\_instances\_offerings
* ec2\_get\_all\_security\_groups
* ec2\_get\_all\_snapshots
* ec2\_get\_all\_spot\_instance\_requests
* ec2\_get\_all\_tags
* ec2\_get\_all\_volume\_status
* ec2\_get\_all\_volumes
* ec2\_get\_all\_zones
* ec2\_get\_console\_output
* ec2\_get\_http\_connection
* ec2\_get\_image
* ec2\_get\_image\_attribute
* ec2\_get\_instance\_attribute
* ec2\_get\_key\_pair
* ec2\_get\_list
* ec2\_get\_object
* ec2\_get\_only\_instances
* ec2\_get\_params
* ec2\_get\_password\_data
* ec2\_get\_path
* ec2\_get\_proxy\_auth\_header
* ec2\_get\_proxy\_url\_with\_auth
* ec2\_get\_snapshot\_attribute
* ec2\_get\_spot\_datafeed\_subscription
* ec2\_get\_spot\_price\_history
* ec2\_get\_status
* ec2\_get\_utf8\_value
* ec2\_get\_volume\_attribute
* ec2\_handle\_proxy
* ec2\_import\_key\_pair
* ec2\_make\_request
* ec2\_modify\_image\_attribute
* ec2\_modify\_instance\_attribute
* ec2\_modify\_network\_interface\_attribute
* ec2\_modify\_reserved\_instances
* ec2\_modify\_snapshot\_attribute
* ec2\_modify\_volume\_attribute
* ec2\_modify\_vpc\_attribute
* ec2\_monitor\_instance
* ec2\_monitor\_instances
* ec2\_new\_http\_connection
* ec2\_prefix\_proxy\_to\_path
* ec2\_proxy\_ssl
* ec2\_purchase\_reserved\_instance\_offering
* ec2\_put\_http\_connection
* ec2\_reboot\_instances
* ec2\_register\_image
* ec2\_release\_address
* ec2\_request\_spot\_instances
* ec2\_reset\_image\_attribute
* ec2\_reset\_instance\_attribute
* ec2\_reset\_snapshot\_attribute
* ec2\_revoke\_security\_group
* ec2\_revoke\_security\_group\_deprecated
* ec2\_revoke\_security\_group\_egress
* ec2\_run\_instances
* ec2\_server\_name
* ec2\_set\_host\_header
* ec2\_set\_request\_hook
* ec2\_skip\_proxy
* ec2\_start\_instances
* ec2\_stop\_instances
* ec2\_terminate\_instances
* ec2\_trim\_snapshots
* ec2\_unassign\_private\_ip\_addresses
* ec2\_unmonitor\_instance
* ec2\_unmonitor\_instances
* ec2\_wait\_for\_state

### SQS Actions

* sqs\_add\_permission.yaml
* sqs\_build\_base\_http\_request.yaml
* sqs\_build\_complex\_list\_params.yaml
* sqs\_build\_list\_params.yaml
* sqs\_change\_message\_visibility.yaml
* sqs\_change\_message\_visibility\_batch.yaml
* sqs\_close.yaml
* sqs\_create\_queue.yaml
* sqs\_delete\_message.yaml
* sqs\_delete\_message\_batch.yaml
* sqs\_delete\_message\_from\_handle.yaml
* sqs\_delete\_queue.yaml
* sqs\_get\_all\_queues.yaml
* sqs\_get\_dead\_letter\_source\_queues.yaml
* sqs\_get\_http\_connection.yaml
* sqs\_get\_list.yaml
* sqs\_get\_object.yaml
* sqs\_get\_path.yaml
* sqs\_get\_proxy\_auth\_header.yaml
* sqs\_get\_proxy\_url\_with\_auth.yaml
* sqs\_get\_queue.yaml
* sqs\_get\_queue\_attributes.yaml
* sqs\_get\_status.yaml
* sqs\_get\_utf8\_value.yaml
* sqs\_handle\_proxy.yaml
* sqs\_lookup.yaml
* sqs\_make\_request.yaml
* sqs\_new\_http\_connection.yaml
* sqs\_prefix\_proxy\_to\_path.yaml
* sqs\_proxy\_ssl.yaml
* sqs\_purge\_queue.yaml
* sqs\_put\_http\_connection.yaml
* sqs\_receive\_message.yaml
* sqs\_remove\_permission.yaml
* sqs\_send\_message.yaml
* sqs\_send\_message\_batch.yaml
* sqs\_server\_name.yaml
* sqs\_set\_host\_header.yaml
* sqs\_set\_queue\_attribute.yaml
* sqs\_set\_request\_hook.yaml
* sqs\_skip\_proxy.yaml
