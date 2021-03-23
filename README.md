# MIDS-251-2021-Final-Project

## S3 Bucket
```
s3://w251-covidx-ct
```

## AWS EC2 Instance
The following steps will be used to start a spot instance in your local terminal. (note: awscli must be setup beforehand)
```
Deep Learning AMI (Ubuntu 18.04) Version 32.0 - ami-0dc2264cd927ca9eb
```
Get your vpcid and create a security group with it
```
aws ec2 describe-vpcs | grep vpcId
aws ec2 create-security-group --group-name fina_project --description "final_project" --vpc-id vpc-b30cd6ce
```
Allow the access to ports 22 and 8888 in the security group for Jupyter Notebook
```
aws ec2 authorize-security-group-ingress --group-id  sg-0d020b3e1af6fbcdf  --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id  sg-0d020b3e1af6fbcdf  --protocol tcp --port 8888 --cidr 0.0.0.0/0
```

Find an instance of our AMI (note: need to have pytorch)
```
aws ec2 describe-images  --filters  Name=name,Values='Deep*Learning*Ubuntu*18.04*32*'
```

Edit the ami, security group, and key name in the code below.  Then start the Deep Learning AMI using a p3.2xlarge instance (which has 8 vcpus and gpu enabled).  Note this will not be the spot instance.
```
aws ec2 run-instances --image-id ami-0dc2264cd927ca9eb \
                      --instance-type p3.2xlarge \
                      --security-group-ids sg-0d020b3e1af6fbcdf \
                      --associate-public-ip-address \
                      --key-name w251-ec2-xavier \
                      --block-device-mapping DeviceName=/dev/sda1,Ebs={VolumeSize=128}
```
For Spot Instance, use below:
```
aws ec2 run-instances --image-id ami-0dc2264cd927ca9eb \
                      --instance-type p3.2xlarge \
                      --security-group-ids sg-0d020b3e1af6fbcdf  \
                      --associate-public-ip-address \
                      --instance-market-options file://spot-options.json \
                      --key-name w251-ec2-xavier \
                      --block-device-mapping DeviceName=/dev/sda1,Ebs={VolumeSize=128}

```

ssh into the instance and then activate pytorch.  Remeber to use your own pem file
```
ssh -i "w251-ec2-xavier.pem" ubuntu@ec2-34-238-192-211.compute-1.amazonaws.com
source activate pytorch_latest_p36
```
## Configure AWS 
AWS needs to be configured to allow communication with s3 bucket where the data will be downloaded
```
aws configure 
Access Key ID:  ####################
Secret Acess Key: ##################
Default region name: us-east-1
Default output format: <hit enter>
```

## Options for retrieving data - numpy files
Images have been matched with their respective labels, randomized, split, and converted to numpy files.  The numpy files may then be converted to tensors for training with pytorch.  In this case, the images do not need to be downloaded.
#### Get from s3 bucket 

Make a folder called data in /home/ubuntu.  Chmod 777 enables copying into the file.
```
mkdir numpy-files
chmod 777 numpy-files
```
Check contents in s3
```
aws s3 ls s3://w251-covidx-ct
```
Copy folder containing the train,hyper, val, and test from the s3 bucket (note: takes about 25 minutes)
```
aws s3 cp s3://w251-covidx-ct/numpy-files/ ~/numpy-files --recursive
```

Copy labels and metadata into data folder
```
aws s3 cp s3://w251-covidx-ct/test_COVIDx_CT-2A.txt .
aws s3 cp s3://w251-covidx-ct/train_COVIDx_CT-2A.txt .
aws s3 cp s3://w251-covidx-ct/val_COVIDx_CT-2A.txt .
aws s3 cp s3://w251-covidx-ct/metadata.csv .
```
## Options for retrieving data - images
Make a folder called data in /home/ubuntu.  Chmod 777 enables copying into the file.
```
mkdir data
chmod 777 data
```
Check contents in s3
```
aws s3 ls s3://w251-covidx-ct
```
Copy folder containing the images from the s3 bucket (note: takes about 25 minutes)
```
aws s3 cp s3://w251-covidx-ct/2A_images/ ~/data/2A_images --recursive
```

Copy labels and metadata into data folder
```
aws s3 cp s3://w251-covidx-ct/test_COVIDx_CT-2A.txt .
aws s3 cp s3://w251-covidx-ct/train_COVIDx_CT-2A.txt .
aws s3 cp s3://w251-covidx-ct/val_COVIDx_CT-2A.txt .
aws s3 cp s3://w251-covidx-ct/metadata.csv .
```

#### Alternatively, get the data from kaggle

Create Kaggle directory
```
mkdir ~/.kaggle
```

Get Kaggle token from Kaggle Account.  Create kaggle.json and copy token into file.
```
cd .kaggle
vim kaggle.json
```

Download Dataset using Kaggle API - 10 minutes 
```
kaggle datasets download -d hgunraj/covidxct
```

Unzip dataset - 10 minutes
```
unzip covidxct.zip
```

## Jupyter Notebook
Once the data has been loaded into the instance, cd into the directory where the data file is located.  Start jupyter notebook using public IP address of instance.  Note don't click into data folder in Jupyter - there are too many files and it may freeze.
```
cd ~
jupyter notebook --ip=0.0.0.0 --no-browser
http://34.238.192.211:8888/?token=856548d1dcecf3200e581fa857396d2568dcacaa7e066c80
```

## Dependencies
```
apt update
apt install python3-pip
apt install unzip

pip3 install kaggle
```




# References

### Get data from Kaggle to S3
https://confusedcoders.com/data-engineering/how-to-copy-kaggle-data-to-amazon-s3

### Create credentials in AWS
https://aws.amazon.com/premiumsupport/knowledge-center/s3-locate-credentials-error/

### s3 bucket commands
https://www.thegeekstuff.com/2019/04/aws-s3-cli-examples/

### Setup Jupyter Notebook (note: not necessary for Deep AMI b/c it comes with anaconda)
https://dataschool.com/data-modeling-101/running-jupyter-notebook-on-an-ec2-server/

### AWS Run Instance commands
https://cloud-gc.readthedocs.io/en/latest/chapter03_advanced-tutorial/advanced-awscli.html
