

## Training (notice I override the entry point)
docker run -it --entrypoint /bin/bash --rm --name placementapp -p 9696:9696 -v $(pwd):/app placementapp
python train.py
cp project_one_model.pkl model/.

# Deployment
Local flask test
Terminal 1: (TODO: dont mount copy weights to image)
docker run -it --rm --name placementapp -p 9696:9696 -v $(pwd):/app placementapp
Termianl 2:
Activate an environment with requests lib
python predict-test.py





Reference: https://github.com/HSubbu/AV-k8s-placement-app