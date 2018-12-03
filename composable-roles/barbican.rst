A few examples for composable roles
###################################

#. OOO ships with quite a few default roles. You can view them using the CLI:

   ::

        openstack overcloud role list

#. If you want to use a custom environment with some of the roles, do the
   following:

   ::

        openstack overcloud roles generate -o my_roles.yaml Controller ObjectStorage

#. Now deploy these roles using by using your custom role definition:

   ::

        openstack overcloud deploy --templates [...] -r custom_roles.yaml --compute-scale 0 --control-scale 1 --swift-storage-scale 1

#. You can also modify the generated YAML file, and remove more service that
   you don't want to be deployed. This is sometimes useful if you're working
   only on a specific service, and just want to test this service. In most
   cases you will need a few additional services like Keystone, MariaDB
   (MySQL), HAProxy. A good starting point is to remove other OpenStack
   services and keeping the remaining services. Have a look at the
   swift_only.yaml for an example.

#. There is another role to prepare an overcloud with just barbican and
   keystone, which is quite nice to do some testing.
  
   First, you'll want to create a barbican-backend-simple-crypto.yaml file which
   also sets the simple crypto backend as the default backendi.  Copy the template
   barbican-backend-simple-crypto.yaml from /usr/share/openstack-tripleo-heat-templates
   to /home/stack, and set BarbicanSimpleCryptoGlobalDefault to true.

   Deploy this with:

   ::

        openstack overcloud deploy --templates \
            -e /usr/share/openstack-tripleo-heat-templates/environments/services/barbican.yaml \
            -e /home/stack/barbican-backend-simple-crypto.yaml -r barbican.yaml

        # Log into the controller node
        nova ssh --network ctlplane --address-type fixed --login heat-admin overcloud-controller-0

#. If you're deploying with tripleo-quickstart, your overcloud-deploy.sh
   command will look as follows::

   openstack overcloud deploy \
       --templates /usr/share/openstack-tripleo-heat-templates \
       --roles-file /home/stack/roles_data_barbican.yaml \
       --libvirt-type qemu --control-flavor oooq_control \
       --compute-flavor oooq_compute --ceph-storage-flavor oooq_ceph \
       --block-storage-flavor oooq_blockstorage \
       --swift-storage-flavor oooq_objectstorage --timeout 90 \
       -e /home/stack/cloud-names.yaml \
       -e /home/stack/containers-default-parameters.yaml \
       -e /usr/share/openstack-tripleo-heat-templates/environments/network-isolation.yaml \
       -e /usr/share/openstack-tripleo-heat-templates/environments/net-single-nic-with-vlans.yaml \
       -e /home/stack/network-environment.yaml \
       -e /home/stack/inject-trust-anchor.yaml \
       -e /usr/share/openstack-tripleo-heat-templates/environments/services/barbican.yaml \
       -e /home/stack/barbican-backend-simple-crypto.yaml \
       --control-scale 1 --compute-scale 0

 #. Test to confirm that barbican and keystone are working.  From the undercloud::

      source ./overcloudrc
      openstack catalog list
      openstack secret list
      openstack secret store --payload "This is my super secret secret"
      openstack secret get --payload ${secret_ref}
