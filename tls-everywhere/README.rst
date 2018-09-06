tls-everywhere lab
==================

It is possible to deploy the overcloud with TLS everywhere enabled, for this,
we use FreeIPA as the CA. The steps to deploy this are in the `TripleO
documentation <http://tripleo.org/install/advanced_deployment/ssl.html#tls-everywhere-for-the-overcloud>`_.

The main idea is:

- Deploy FreeIPA on a separate node.
- Enroll the undercloud to FreeIPA and give it the appropriate
  privileges/permissions.
- Deploy novajoin
- Deploy the overcloud with the appropriate environment files.

Using quickstart, this is already automated for you. So, if you want to deploy
a queens environment for quickstart, you need to run::

    ./quickstart.sh --no-clone --teardown all --clean \
        -p quickstart-extras.yml -N config/nodes/1ctlr_1comp_1supp.yml \
        -c config/general_config/ipa.yml \
        -R queens --tags all $VIRTHOST

.. NOTE:: You might want to add the ``-w`` option to pass a specific directory
          for this environment.
