JupyterHub (OpenShift Authenticator)
====================================

This repository contains an example of a customised deployment of JupyterHub for OpenShift, which uses the same OpenShift cluster as the deployment is running in, as the authentication provider. It uses the ``jupyterhub`` image for running JupyterHub on OpenShift as an S2I builder to create a customised image containing the support for the OpenShift authenticator.

Building Required Images
------------------------

To use this example you need to build the Jupyter notebook images from the [jupyter-notebooks](https://github.com/jupyter-on-openshift/jupyter-notebooks) repository and the JupyterHub image from the [jupyterhub-quickstart](https://github.com/jupyter-on-openshift/jupyterhub-quickstart) repository. These can be created by running:

```
oc create -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyter-notebooks/master/images.json
oc create -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyterhub-quickstart/master/images.json
```

Loading the Templates
---------------------

There is no need to load the templates from the other repositories just to see this example in action. You do need to load the templates from this repository though. Run:

```
oc create -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyterhub-ocp-oauth/master/templates.json
```

This will create the following templates:

```
jupyterhub-ocp-oauth
```

Deploying the Example
---------------------

Once the images above have finish building, it deploy the example, run:

```
oc new-app --template jupyterhub-ocp-oauth
```

The name of the deployment created will be ``jupyterhub``. If you want to change the name, instead run:

```
oc new-app --template jupyterhub-ocp-oauth \
  --param APPLICATION_NAME=my-jupyterhub
```

Note that in this example there is no need to pre-create any service account or setup roles as the template will create these for you.

Once JupyterHub has finished deploying open it from your browser. You should be prompted to login using OpenShift. Accept use of your OpenShift account by the application and you should be able to start your Jupyter notebook server.

Deleting the Application
------------------------

To delete the JupyterHub instance along with all notebook instances, run:

```
oc delete all,configmap,pvc,serviceaccount,rolebinding --selector app=my-jupyterhub
```

This command is a bit different to deleting the JupyterHub instance when using the [jupyterhub-quickstart](https://github.com/jupyter-on-openshift/jupyterhub-quickstart) repository. This is because the templates in this example create a service account and role binding unique to the application so they also need to be deleted.

How does this Example Work
--------------------------

This repository contains two key files:

* [jupyterhub_config.py](./jupyterhub_config.py)
* [requirements.txt](./requirements.txt)

The ``jupyterhub_config.py`` file extends the JupyterHub configuration to enable use of the OpenShift authenticator. This relies on the template having added necessary annotations to the service account created to permit use of the service account as an OAuth provider.

The code in the ``jupyerhub_config.py`` file will then automatically configure JupyterHub with the necessary OAuth client ID, secret, and public callback URL, by querying the OpenShift environment for the details. There is no need for any manual configuration.

The ``requirements.txt`` file lists additional Python packages that are required by the code in ``jupyterhub_config.py``.

When the application is created from the template, a build will be setup that runs the ``jupyterhub`` image as an S2I builder on the code from this repository. The result of that will be that the additional Python packages listed in the ``requirements.txt`` file will be installed into the image and the ``jupyterhub_config.py`` copied into an appropriate location.

The customised JupyterHub image generated by the S2I build will then be what is deployed. It will run JupyterHub and the ``jupyterhub_config.py`` from this repository will be merged with the default configuration and any further configuration you might supply via the ``JUPYTERHUB_CONFIG`` template parameter.

Using the OpenShift Web Console
-------------------------------

JupyterHub can be deployed using this template from the web console by selecting _Select from Project_ from the _Add to Project_ drop down menu and choosing _JupterHub (OCP OAuth)_.
