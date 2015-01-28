based on: https://github.com/veverjak/coreos-mesos-marathon

We create a VM for hosting Babelnet noise detector. Once installed, test the API at the port 9990 of localhost.

# Pull playbooks
git subtree pull --prefix ansible/roles/ansible-coreos-babelnet-hujo https://github.com/ptorrestr/ansible-coreos-babelnet-hujo master

# Prerequisites

- fleetctl
- vagrant
- virtualbox
- python
 - docker-py==0.3.1
 - ansible (pip install ansible==1.5)

# Install

 - Start CoreOS cluster
  - Vagrant test: ``cd vagrant; vagrant up``
 - Connect to fleet from localhost - https://github.com/coreos/fleet/blob/master/Documentation/remote-access.md
  - fleet is getting ssh-keys from ssh-agent so if you don't have yours there already you can add one using
  - ``eval `ssh-agent -s` ``
  - ``ssh-agent && vagrant ssh-config core-01 | sed -n "s/IdentityFile//gp" | xargs ssh-add``
  - ``export FLEETCTL_TUNNEL="$(vagrant ssh-config core-01 | sed -n "s/[ ]*HostName[ ]*//gp"):$(vagrant ssh-config core-01 | sed -n "s/[ ]*Port[ ]*//gp")"``
  - ``export DOCKER_HOST=tcp://localhost:2375``
 - Remove fleetctl known\_host file and clean the ssh known\_host file
  - ``rm ~/.fleetctl/known_hosts`` 
 - Setup your hujo-account on ansible/group\_vars/all
 - make initial changes on coreos hosts, following commnads have to be run from ansible directory
  - make sure that ansible is in the path.
  - ansible-playbook -i vagranttest coreos.yml
 - submit and start following units:
  - registry
   - ansible-playbook -i vagranttest coreos-registry.yml
  - wildfly
   - ansible-playbook -i vagranttest coreos-wildfly.yml
  - babelnet-hujo
   - ansible-playbook -i vagranttest coreos-babelnet-hujo.yml
 - Test babelnet API
   - `` curl -X POST http://localhost:9990/babelnet/filter/tweets -H "Content-Type: application/json" -d '{"tweets":[{"id":1,"text":"my text"}, {"id":2,"text":"Q for booksellers: if I put the barcode on the *inside* back flap of a dust jacket, am I shooting myself, or you, in the foot?"}]}'
     ``

#Add new playbooks:
git subtree add --prefix ansible/roles/{name} https://github.com/ptorrestr/{name} master --squash

#Update playbooks:
git subtree pull --prefix ansible/roles/{name} https://github.com/ptorrestr/{name} master --squash

#Debugging
Add this text for the variable in ansible
``
- name: Display variable
  debug: var=variable
``

