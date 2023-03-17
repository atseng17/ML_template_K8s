

## Training (notice I override the entry point)
cd ml-dev
docker run -it --entrypoint /bin/bash --rm --name placementapp -p 9696:9696 -v $(pwd):/app placementapp
python train.py
cp project_one_model.pkl ../ml-dev/.
# Deployment
Local flask test
Terminal 1:
cd app
docker run -it --rm --name placementapp -p 9696:9696 placementapp

Terminal 2:
cd app
Activate an environment with requests lib
python predict-test.py





Reference: https://github.com/HSubbu/AV-k8s-placement-app