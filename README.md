# concourse-windows-release

## Bosh deployment manifest

```
---
name: concourse-windows-worker

releases:
- name: concourse-windows-worker
  version: 2.7.3

# Uses the stemcell available here:
#  https://bosh.io/stemcells/bosh-aws-xen-hvm-windows2012R2-go_agent
stemcells:
- alias: windows
  os: windows2012R2
  version: latest

update:
  canaries: 0
  canary_watch_time: 60000
  update_watch_time: 60000
  max_in_flight: 2

instance_groups:
- name: gpdb5-windows-worker
  instances: 2
  vm_type: worker
  stemcell: windows
  azs: [z1]
  networks: [{name: internal}]
  jobs:
    - name: concourse_windows
      templates:
        - name: concourse_windows
      properties:
        concourse_windows:
          tags: ["gpdb5-windows-worker"]
          tsa_host: gpdb.ci.pivotalci.info
          tsa_public_key: |
            ssh-rsa --Redacted-- pa-toolsmiths@pivotal.io
          tsa_worker_private_key: |
            -----BEGIN RSA PRIVATE KEY-----
                    --Redacted--
            -----END RSA PRIVATE KEY-----
```

## Relevent sections of a bosh cloud config

```
azs:
- name: z1
  cloud_properties: {availability_zone: us-west-2a}

vm_types:
- name: worker
  cloud_properties:
    instance_type: c4.2xlarge
    ephemeral_disk: {size: 200000, type: gp2}

networks:
- name: internal
  type: manual
  subnets:
  - range: 10.0.0.0/20
    reserved:
      - 10.0.0.1 - 10.0.0.4
    gateway: 10.0.0.1
    az: z1
    dns: [10.0.0.2]
    cloud_properties:
      subnet: {subnet id}
      security_groups: {security group id}

compilation:
  workers: 5
  reuse_compilation_vms: true
  az: z1
  vm_type: worker
  network: internal
```

## Example concourse pipeline

```
---
jobs:
- name: test
  plan:
  - task: test
    tags: ['gpdb5-windows-worker']
    config:
      platform: windows
      run:
        path: 'whoami'
        args: ['/all']
```

