# aws_manage_vpc_peer

## Description:

Manages AWS Virtual Private Clouds (VPCs) Peering connections as follows:
- Running the role will create the VPC peering connections defined in `aws_vpc_peers` when the variable `arg_action` is not defined or is defined and set to `create`
- Running the role will delete the VPC peering connections defined in `aws_vpc_peers` when the variable `arg_action` is defined and is set to `delete` 
- Running the [get_vpc_peering_connection_facts.yml](./tasks/get_vpc_peering_connection_facts.yml) play will return information about the specified VPC peering connection.

## Behaviour:

**Feature:** Create AWS VPC peer
- **Given** valid AWS credentials
- **Given** two existing AWS VPCs
- **When** the script is executed
- **Then** a VPC peer is created and accepted

## Configuration:

Common variables used by this role:

| Variable | Description | Example |
|---|---|---|
| **aws_region** | AWS region | See [AWS regions](http://docs.aws.amazon.com/general/latest/gr/rande.html#ec2_region) |
| **aws_access_key** | AWS access key | AKIAIOSFODNN7EXAMPLE |
| **aws_secret_key** | AWS secret key | wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY |
| **resource_tags** | List of project tags | see usage |

Variables specific to this role:

| Variable | Description | Example |
|---|---|---|
| **aws_vpc_peers** | List of VPC peers to create | see usage |
| **vpc_pc_tag** | The `Name:` tag of the peering connection to get information about | see usage |

## Usage:

How to invoke the role from a playbook (create peering connections):

```yaml
- name: Create AWS VPC peer
  include_role:
    name: aws_manage_vpc_peer
  vars:
    aws_access_key: AKIAIOSFODNN7EXAMPLE
    aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    aws_region: eu-west-2  # London

    resource_tags:
      Name: "{{ resource_name }}"   # Set to peering connection tag
      Owner: "Acme Co."

    aws_vpc_peers:
      - aws_vpc_tag: "Test A"
        aws_vpc_peer: "Test B"
        aws_vpc_region: "{{ aws_region }}"
        aws_vpc_pc_tag: "Test A-B"
        aws_vpc_pc_tags: "{{ resource_tags }}"
```

How to invoke the role from a playbook (delete peering connections):

```yaml
- name: Create AWS VPC peer
  include_role:
    name: aws_manage_vpc_peer
  vars:
    arg_action: "delete"
    aws_access_key: AKIAIOSFODNN7EXAMPLE
    aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    aws_region: eu-west-2  # London

    resource_tags:
      Name: "{{ resource_name }}"   # Set to peering connection tag
      Owner: "Acme Co."

    aws_vpc_peers:
      - aws_vpc_tag: "Test A"
        aws_vpc_peer: "Test B"
        aws_vpc_region: "{{ aws_region }}"
        aws_vpc_pc_tag: "Test A-B"
        aws_vpc_pc_tags: "{{ resource_tags }}"
```

How to get VPC peering connection information:

```yaml
- name: Get peering connection information
  include_role:
    name: aws_manage_vpc_peer
    tasks_from: get_vpc_peering_connection_facts
  vars:
    vpc_pc_tag: "Test A"
```

This will return facts in the variable `vpc_peering_facts`.  The data returned is as follows:

```yaml
"vpc_peering_facts": {
    "changed": false, 
    "failed": false, 
    "result": [
        {
            "accepter_vpc_info": {
                "cidr_block": "10.11.0.0/16", 
                "cidr_block_set": [
                    {
                        "cidr_block": "10.11.0.0/16"
                    }, 
                    {
                        "cidr_block": "10.22.0.0/16"
                    }, 
                    {
                        "cidr_block": "10.33.0.0/16"
                    }
                ], 
                "owner_id": "121212121212", 
                "peering_options": {
                    "allow_dns_resolution_from_remote_vpc": false, 
                    "allow_egress_from_local_classic_link_to_remote_vpc": false, 
                    "allow_egress_from_local_vpc_to_remote_classic_link": false
                }, 
                "region": "eu-west-2", 
                "vpc_id": "vpc-8fd11be7"
            }, 
            "requester_vpc_info": {
                "cidr_block": "10.44.0.0/22", 
                "cidr_block_set": [
                    {
                        "cidr_block": "10.44.0.0/22"
                    }, 
                    {
                        "cidr_block": "10.55.0.0/22"
                    }, 
                    {
                        "cidr_block": "10.66.0.0/22"
                    }
                ], 
                "owner_id": "121212121212", 
                "peering_options": {
                    "allow_dns_resolution_from_remote_vpc": false, 
                    "allow_egress_from_local_classic_link_to_remote_vpc": false, 
                    "allow_egress_from_local_vpc_to_remote_classic_link": false
                }, 
                "region": "eu-west-2", 
                "vpc_id": "vpc-ecd61c84"
            }, 
            "status": {
                "code": "active", 
                "message": "Active"
            }, 
            "tags": [
                {
                    "key": "Owner", 
                    "value": "Acme Co."
                }, 
                {
                    "key": "Name", 
                    "value": "Test A-B"
                }
            ], 
            "vpc_peering_connection_id": "pcx-85bb27ec"
        }
    ]
}
```