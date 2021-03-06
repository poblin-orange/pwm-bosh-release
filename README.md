# BOSH Release for PWM

PWM is an open source password self service application for LDAP directories. PWM is an ideal candidate for organizations that wish to “roll their own” password self service solution, but do not wish to start from scratch. [Overview/Screenshots](https://docs.google.com/presentation/d/1LxDXV_iiToJXAzzT9mc1xXO0atVObmRpCame6qXOyxM/pub?slide=id.p8)

Official project page is at [https://github.com/pwm-project/pwm/](https://github.com/pwm-project/pwm/).


The purpose of this bosh release is to provide Bosh operators an easy solution for self service onboarding.
Typical use is managing identity in an OpenLDAP directory, and configure Cloudfoundry UAA to use OpenLDAP as identity backend


This release brings the following feature:
* give pwm operators all the bosh benefits
** OS update with Bosh stemcell mechanisms
** java jdk and tomcat versionning and update with Bosh release mechanism
** multi-Iaas, multi-Os capability
** clean separation of data (Bosh persistent disk), logs and configuration
** unified operation logic for Cloudfoundry / Bosh operators
** pwm configuration for declarative bosh manifest file


for best practices configuring :
* http://blog.trifork.com/2015/07/28/setting-up-pwm-as-a-password-recovery-tool-for-openldap/


Main built-in configuration choices
* openldap backend
* disabled http session persistence
* session management LOCAL
* expects front end https offload (ie go router, with route registrar in cloudfoundry context)

see extensive pwm reference docs for evolution
* http://www.pwm-project.org/pwm/public/reference/settings.jsp#settings_category_NOTES


For convenience and test purpose, can use a openldap docker image, inside a bosh docker release.
* https://github.com/osixia/docker-openldap
* https://github.com/cloudfoundry-community/docker-boshrelease




## Usage

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/cloudfoundry-community/pwm-bosh-boshrelease.git
cd pwm-bosh-boshrelease
bosh upload release releases/pwm-bosh-1.yml
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest & deploy a cluster:

```
templates/make_manifest warden
bosh -n deploy
```

For AWS EC2, create a single VM:

```
templates/make_manifest aws-ec2
bosh -n deploy
```

### Override security groups

For AWS & Openstack, the default deployment assumes there is a `default` security group. If you wish to use a different security group(s) then you can pass in additional configuration when running `make_manifest` above.

Create a file `my-networking.yml`:

``` yaml
---
networks:
  - name: pwm-bosh1
    type: dynamic
    cloud_properties:
      security_groups:
        - pwm-bosh
```

Where `- pwm-bosh` means you wish to use an existing security group called `pwm-bosh`.

You now suffix this file path to the `make_manifest` command:

```
templates/make_manifest openstack-nova my-networking.yml
bosh -n deploy
```

### Development

As a developer of this release, create new releases and upload them:

```
bosh create release --force && bosh -n upload release
```

### Final releases

To share final releases:

```
bosh create release --final
```

By default the version number will be bumped to the next major number. You can specify alternate versions:


```
bosh create release --final --version 2.1
```

After the first release you need to contact [Dmitriy Kalinin](mailto://dkalinin@pivotal.io) to request your project is added to https://bosh.io/releases (as mentioned in README above).
