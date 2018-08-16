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
