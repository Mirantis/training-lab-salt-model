# Training Lab - salt model

salt model used in "training-lab": https://github.com/Mirantis/training-lab

## How to create the salt model

```
OUTPUT_DIRECTORY="/tmp/salt-model"
COOKIECUTTER_TEMPLATE="$HOME/data/github/training-lab-salt-model/salt-model_cookiecutter-context"
MCP_RELEASE="2018.8.0"

cd /tmp/
git clone ssh://pruzicka@gerrit.mcp.mirantis.net:29418/mk/cookiecutter-templates
cd cookiecutter-templates
git checkout $MCP_RELEASE

mkdir -p $OUTPUT_DIRECTORY/classes/cluster $OUTPUT_DIRECTORY/nodes
python generate.py --config-file $COOKIECUTTER_TEMPLATE --template cluster_product/infra --output-dir $OUTPUT_DIRECTORY/classes/cluster
python generate.py --config-file $COOKIECUTTER_TEMPLATE --template cluster_product/cicd --output-dir $OUTPUT_DIRECTORY/classes/cluster
python generate.py --config-file $COOKIECUTTER_TEMPLATE --template cluster_product/openstack --output-dir $OUTPUT_DIRECTORY/classes/cluster

cat > $OUTPUT_DIRECTORY/nodes/cfg01.tng.mirantis.com.yml << EOF
classes:
- cluster.training-lab.infra.config
parameters:
  _param:
    linux_system_codename: xenial
    reclass_data_revision: master
  linux:
    system:
      name: cfg01
      domain: tng.mirantis.com
EOF

cd $OUTPUT_DIRECTORY
git init
git add -A
git commit -m "Create model training-lab"

git submodule add https://github.com/Mirantis/reclass-system-salt-model.git classes/system
git -C classes/system checkout $MCP_RELEASE
git add -A
git commit -m "Added new shared reclass submodule"
```

## Check model

Based on: https://github.com/Mirantis/pipeline-library/blob/master/src/com/mirantis/mk/SaltModelTesting.groovy

```
docker run --rm -it -u root:root -v $PWD:/salt-model -w /salt-model mirantis/salt:saltstack-ubuntu-xenial-salt-2017.7 bash

CLUSTER_NAME="testcluster"
DEBUG=1
FORMULAS_SOURCE=pkg
MASTER_HOSTNAME="salt"
MINION_ID="salt"
RECLASS_VERSION="master"
SALT_FORMULAS_REVISION="stable"
SALT_STOPSTART_WAIT=5

cat > /etc/apt/sources.list << EOF
deb [arch=amd64] http://mirror.mirantis.com/$SALT_FORMULAS_REVISION/ubuntu xenial main restricted universe
deb [arch=amd64] http://mirror.mirantis.com/$SALT_FORMULAS_REVISION/ubuntu xenial-updates main restricted universe
EOF

apt update
apt install -y python-netaddr

echo '127.0.1.2 salt' >> /etc/hosts

git clone https://github.com/salt-formulas/salt-formulas-scripts /srv/salt/scripts
rsync -ah . /srv/salt/reclass


for PYTHON_DIST_PACKAGES_DIR in $(python -c "import site; print(' '.join(site.getsitepackages()))"); do
  pip install --install-option="--prefix=" --upgrade --force-reinstall -I -t "$PYTHON_DIST_PACKAGES_DIR" git+https://github.com/salt-formulas/reclass.git@${RECLASS_VERSION};
done

source /srv/salt/scripts/bootstrap.sh
cd /srv/salt/scripts
source_local_envs
configure_salt_master
configure_salt_minion
install_salt_formula_pkg
saltservice_restart
source_local_envs
saltmaster_init
verify_salt_minions
```

