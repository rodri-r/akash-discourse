# akash-discourse
This will deploy a Discourse forum on Akash

# Setup an "ssh" ubuntu image on [ Akash ](https://akash.network)  

## Optional : build custom Docker image (recommended)
Update Dockerfile with your needs.
```
# Note you can also use the prebuilt image at moonbys/moonubu:super1
$ docker build -t $your_dockerhub_user/ubuntu:1 .
```
Update deploy.yaml with your newly built docker image. ( or keep the prebuilt image)
## Buy me a coffee to keep the work going

| | |
--- | --- 
|akash:|`comingsoon`|

## Prepare your ssh session

On your workstation setup an ssh public/private key pair.
```
$ssh-keygen -t rsa
# (a password is optional. But in most cases you don't need one).
# Note that $HOME/.ssh/id_rsa.pub is now created
```

## Update deploy.yaml

Copy and paste the contents of `$HOME/.ssh/id_rsa.pub` and paste them into deploy.yaml (env -> pubkey). (deploy.yaml : line 8).

NOTE: Make sure that you do not use any `"` around the pubkey as this will cause problems.


### Update resources as needed. Set the amount of RAM / CPU / diskspace as needed.

## Deploy to Akash
* This assumes that you have installed akash in `$HOME/bin/akash`
* Read through https://docs.akash.network & https://github.com/tombeynon/akash-deploy. Thanks for the excellent guides!
* Broadly speaking you will need to:

###  Create an akash address 
``` 
make address
echo "export AKASH_ADDRESS=**address** " >> env.sh
``` 

** Fund `AKASH_ADDRESS` with at least 5 AKT
** create a deployment (you can use deploy.yaml) as an example
```
# Need to do this once
make create_certificate
```
```
make create_deployment
make list_bid
```
### accept a bid
```
make create_lease PROVIDER=**provider** DSEQ=**dseq**
```
### push the manifest
```
make send_manifest PROVIDER=**provider** DSEQ=**dseq**
```
### Read logs
```
make lease_logs PROVIDER=**provider** DSEQ=**dseq**
```

## Most importantly -- how do you interact with this?

```
make lease_status PROVIDER=**provider** DSEQ=**dseq**
```

Take a note of the URL and exposed random port. That is your ssh port to access.

```
ssh -p $RANDOMPORT root@URL
```

### Debug

If things are still not working you can set "debug=true" in the environment variable section of deploy.yaml. This will cause sshd to run with debug mode to get additional info. As a side effect as soon as you close your session it will kill the SSH session.


# Preparing your Ubuntu install

## Install Ruby and Ruby Version Manager

```
command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
```
```
command curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -
```
```
curl -sSL https://get.rvm.io | bash -s stable
```
```
echo 'gem: --no-document' >> ~/.gemrc
```
```
source /usr/local/rvm/scripts/rvm
```
```
rvm --version
```

## Upgrade Ruby, set default version & install

```
rvm install 2.6.2
```
```
rvm --default use 2.6.2
```
```
gem install bundler mailcatcher rake
```

## Start Postgresql server and setup database

```
service postgresql start
```

### login to postgres cli

```
sudo -u postgres -i
```
### create database, create user and make them a superuser

```
psql template1
```
```
CREATE USER youruser WITH PASSWORD 'yourpassword';
```
```
CREATE DATABASE discourse_development;
```
```
GRANT ALL PRIVILEGES ON DATABASE discourse_development to youruser;
```
```
ALTER USER youruser WITH SUPERUSER;
```

### confirm and exit

```
\du
```
```
\q
```
```
exit
```

## Install NVM & Node

### NVM

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```

### Setup path

```
export NVM_DIR="$HOME/.nvm"
```

### Check path works

```
nvm --version
```
### Install Node & set default version

```
nvm install node
```
```
nvm alias default node
```
```
npm install -g svgo
```

# Install Discourse

```
git clone https://github.com/discourse/discourse.git ~/discourse
```
```
cd ~/discourse
```
```
apt-get install libpq-dev
```
```
bundle install
```

### Check postgres server status

```
service postgresql status
```
### If it is not running, start it

```
service postgresql start
```

### Run Redis Server in the background

```
redis-server --daemonize yes
```

### Create the database and run migrations

```
bundle exec rake db:create
```
```
bundle exec rake db:migrate
```
```
RAILS_ENV=test bundle exec rake db:create db:migrate
```

###Create an admin account with:

```
bundle exec rake admin:create
```
#### Follow the steps for setting up admin email and password

## FINALLY

### Launch Discourse on Akash

```
bundle exec rails s -b 0.0.0.0 
```
### open browser on http://localhost:3000 and you should see Discourse
