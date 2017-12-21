# postgresql
community postgresql release
## configuration
If any pairs don't apply, comment them out.
### aws
Certain _key:value_ pairs can be configured per project. Others should remain fixed for all projects.

These values should remain unchanged.

    cloud: aws
Replace TENANT with the name of the AWS_PROFILE environment variable. 
    
    tenant: TENANT
Replace PROJECT_NAME with the name of the project hosting the instances. 

    project: PROJECT_NAME
Replace DOMAIN with a `production` or `development` (or another value of your choosing).

    domain: DOMAIN
These values should remain unchanged.
    
    application: postgresql
Replace REGION with your desired AWS region such as `us-east-1` or `ap-southeast-2`.

    region: REGION    
The following settings are optional. If not defined, they will be replaced with the string `none`.

    cluster: a
    role: none
    dataflow: none
Replace REPLICA with a `yes` or `no` to indicate whether you want a standalone server or master/replica pair.

    replica: yes
Replace USER with `centos` for CentOS or `redhat` for RHEL.

    user: USER
Replace AMI with the AMI ID in the region you wish to deploy; Centos 7.3 or RHEL 7.3 are supported. If commented out or removed, CentOS 7.3 will be booted.

    image: AMI
Replace SIZE with  `t2.micro`, `t2.small`, `t2.medium`, `t2.large`, `t2.xlarge`, `t2.x2large`,  or another value from [here](https://aws.amazon.com/ec2/instance-types/).

    type: SIZE
Replace SIZE with the size of the root or data volume in GB, e.g, `20` for 20gig or `2000` for 2TB. `11` is a reasonable value for the root volume.

    root_volume: SIZE
    data_volume: SIZE
Replace CIDR_BLOCK and SUBNET with the values in the VPC, e.g., 172.31.0.0/16, 172.31.1.0/24, 172.31.2.0/24. The internal subnet is private and non-routable, the external subnet will outbound routing.
    
    cidr_block: CIDR
    internal_subnet: SUBNET
    external_subnet: SUBNET
(if needed) Replace  PROXY with `yes` or `no` if there is an HTTP proxy involved in getting to the internet.
 
    http_proxy: http://USER:PASSWORD@DOMAIN:PORT
    
### osp
Certain _key:value_ pairs can be configured per project. Others should remain fixed for all projects.

These values should remain unchanged.

    cloud: osp
Replace TENANT with the name of the configure OSP tenant. 

    tenant: TENANT
Replace PROJECT_NAME with the name of the project hosting the instances. 
    
    project: PROJECT
Replace DOMAIN with a `production` or `development` (or another value of your choosing).

    domain: DOMAIN
These values should remain unchanged.

    application: postgresql
Replace REGION with your configured OSP region.

    region: OSP_REGION
The following settings are optional. If not defined, they will be replaced with the string `none`.

    cluster: a
    role: none
    dataflow: none
Replace REPLICA with a `yes` or `no` to indicate whether you want a standalone server or master/replica pair.

    replica: REPLICA
Replace USER with `centos` for CentOS or `redhat` for RHEL.

    user: USER
Replace IMAGE_UUID with the UUID of the image you wish to boot.

    image: IMAGE_UUID
Replace SIZE with  configured virtual machine size.

    type: TYPE
Replace SIZE with the size of the root or data volume in GB, e.g, `20` for 20gig or `2000` for 2TB. `11` is a reasonable value for the root volume. If booting from an image, this is ignored.

    root_volume: SIZE
    data_volume: SIZE
Replace POOL with any configured block pool, e.g., ScaleIO or Ceph.

    block_pool: POOL
Replace  SUBNET with the configured values, e.g., 172.31.1.0/24, 172.31.2.0/24. The internal subnet is private and non-routable, the external subnet will outbound routing.
    
    internal_subnet: SUBNET
    external_subnet: SUBNET
Replace  FLOAT with the floating IP pool, e.g., external. 
    
    float_pool: FLOAT
Replace SUBNET_UUID with the UUID's of the internal and external subnets.

    internal_uuid: SUBNET_UUID
    external_uuid: SUBNET_UUID
Replace ZONE with the configured OSP zone.

    zone: ZONE 
(if needed) Replace USER, PASSWORD, DOMAIN, PORT with the necessary values.

    http_proxy: http://USER:PASSWORD@DOMAIN:PORT
    
## deployment
### amazon web services
You will  want to set your AWS_PROFILE to your preferred credentials.

Assuming `aws.yml` is the name of the inventory configuration file.  To deploy the postgresql overlay:

    ./deploy aws.yml
    
To deploy both a VM and postgresql overlay:

    ./deploy full aws.yml

Expected running times:

* standlone approximately 12 minutes 
* master/replica approximately 19 minutes

### openstack
Assuming `osp.yml` is the name of the inventory configuration file.  To deploy the postgresql overlay:

    ./deploy osp.yml
    
To deploy both a VM and postgresql overlay:

    ./deploy full osp.yml

Expected running times:

* standlone approximately 12 minutes 
* master/replica approximately 16 minutes

## testing
If the `deploy` code ran without errors, that's a pretty good indicator of a successful deployment. However, you can run the following to be sure:

To test either a standalone or cluster deployment with the configuration of `aws.yml`:

    ./test aws.yml
        
### expected results
Depending on your configuration (standalone vs replica), you will see various skipped plays, this is normal. Ignore the majority of the output until the end when you'll see the following:

     "msg": [
            "SELECT 500"
        ]

This indicates the `t_random` table was successfully created on the standalone or master. If you've also created a replica, look for something similar that follows:

    "msg": [
            " s  |               md5                ",
            "----+----------------------------------",
            "  1 | a5b6eb6b8f14a2a4df2df955031f2ba7",
            "  2 | 0a40ce5c1f2065dabf1cf0c1b7aa8979",
            "  3 | 66fc054c33e17c8f68c721685db3cc6e",
            "  4 | b562fa0f98a6b078bf33c1772d9e5370",
            "  5 | bd8324ae639812ba9efa09be91b5c5cb",
            "  6 | 9dd06419a085b55557f7e9ccdf7f152a",
            "  7 | 202b558c6dde99b40e4dedb7542ed4e5",
            "  8 | 0048168d4601c71d53846d38a9a80611",
            "  9 | a51e388950257283c7fe62508fdfb299",
            " 10 | a9814e339e7f93c1fab22c808844dc13",
            " 11 | 653aded1ffb1a1c6276f5d6426306496",
            " 12 | c828adde266bdec9963f83bccdaf731f",
            " 13 | 14c3e9a03cb6e33676bdfe9a4dd143c0",
            " 14 | 895db8a45c71134009da8e1095c857db",
            " 15 | 9710d6a6f894355520dcd329ce761c89",
            "(15 rows)"
        ]

This shows the select was successfully run on the replica, indicating a successful deployment. Next, you'll see:

    "msg": [
            "DROP TABLE"
        ]
        
This shows the successful  removal of the test `t_random` table from the standalone (or master which will also stream the removal to the replica).
