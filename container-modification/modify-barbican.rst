Modifying a Docker container in TripleO (barbican)
##################################################

#. Let's try to modify a barbican container to test out a simple change.

   Start off by patching the barbican srpm
   ::

       sudo yum install rpm-build
       sudo yum install  http://download.eng.bos.redhat.com/brewroot/packages/python-pykmip/0.5.0/1.el7ost/noarch/python2-pykmip-0.5.0-1.el7ost.noarch.rpm
       curl -O http://download.eng.bos.redhat.com/brewroot/packages/openstack-barbican/7.0.0/0.20180830222330.3f6ccca.el7ost/src/openstack-barbican-7.0.0-0.20180830222330.3f6ccca.el7ost.src.rpm
       rpm -ivh openstack-barbican*
       sudo yum-builddep ~/rpmbuild/SPECS/openstack-barbican.spec
       cd ~/rpmbuild/SOURCES
       curl -O https://github.com/JAORMX/tripleo-workshop/blob/master/container-modification/barbican.patch

       Modify the spec file to include the patch with the -p2 otion.
       
       rpmbuild -ba ~/rpmbuild/SPECS/openstack-barbican.spec

#. Create a new docker file to remove the old rpms and add the new rpms.  The dockerfile will
   look something like this,  Check the rpm names and the image name. 
   ::

        FROM 192.168.24.1:8787/tripleomaster/centos-binary-barbican-api:current-tripleo

        USER root
        RUN yum remove -y openstack-barbican-api openstack-barbican-common python-barbican
        ADD python-barbican-7.0.0-0.20180801155935.08ca228.el7.noarch.rpm /opt
        ADD openstack-barbican-common-7.0.0-0.20180801155935.08ca228.el7.noarch.rpm /opt
        ADD openstack-barbican-api-7.0.0-0.20180801155935.08ca228.el7.noarch.rpm /opt
        RUN rpm -i /opt/python-barbican-7.0.0-0.20180801155935.08ca228.el7.noarch.rpm
        RUN rpm -i /opt/openstack-barbican-common-7.0.0-0.20180801155935.08ca228.el7.noarch.rpm
        RUN rpm -i /opt/openstack-barbican-api-7.0.0-0.20180801155935.08ca228.el7.noarch.rpm
        USER barbican

#. Build the new container and push it to the local registry. 
   ::

       sudo docker build -t 192.168.24.1:8787/tripleomaster/centos-binary-barbican-api:rickroll -f ./Dockerfile-api .
       sudo docker push 192.168.24.1:8787/tripleomaster/centos-binary-barbican-api:rickroll

#. Now use the modified container by deploying with the additional environment
   file.  Modify the environment file to reference the Barbican container:

   ::

        openstack overcloud deploy --templates [...] -e use-modified-container.yaml

#. To test out this patch, create a secret and check the barbican logs.
   Look for XXXX. 
