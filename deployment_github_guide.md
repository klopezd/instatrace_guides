## Deployment Guide from Github on EFW

### Creating a new release folder and cloning for the first time

**1. Create a new folder inside the releases folder**
* _/var/www/instance/_

```
mkdir YYYYMMddHHmmss
```

**2. Navigate to the folder and clone the git repository**
* _/var/www/instance/releases/_

```
cd YYYYMMddHHmmss
git clone https://github.com/instatracellc/web-instatrace-efw.git .
```

**3. Create the symbolic links to the shared files**
* _/var/www/instance/releases/YYYYMMddHHmmss_

* If any of the files exists on the releases/YYYYMMddHHss folder must be deleted first and then created (linked) to the shared folder content
 
```
ln -s /var/www/efw/shared/log log
ln -s /var/www/efw/shared/config/database.yml config/database.yml
ln -s /var/www/efw/shared/config/environments config/environments
ln -s /var/www/efw/shared/config/shards.ymlg config/shards.yml
ln -s /var/www/efw/shared/config/initializers/init_vars.rb config/initializers/init_vars.rb
```
*Folder actions
```
#Remove
rm -rf folderName

#Copy
 cp -R SorucefolderName /var/www/Destination
```

**4. Activate the Ruby version**
* _/var/www/instance/releases/YYYYMMddHHmmss_

```
 cd /var/www/efw/current

 rvm --default use ruby-2.2.2

 gem install bundler --no-rdoc --no-ri

# install bundle dependencies
# production
 bundle install --deployment --without development test
# development/test
 bundle install --no-deployment 
```

**5. Setup secret key**
* _/var/www/instance/releases/YYYYMMddHHmmss_

```
bundle exec rake secret
vim config/secrets.yml

# /var/www/efw/instatrace/config/secrets.yml
production:
  secret_key_base: [here the value that you copied from 'rake secret']
```

**6. Precompile the assets and public links**
*  _/var/www/instance/releases/YYYYMMddHHmmss_

* Right at the beginning of the function *
```
bundle exec rake assets:precompile RAILS_ENV=environment
```

*Create the NFS symbolic link to arch *
```
ln -s  /var/www/efw/arch public/uploads
```

**7. IF NOT IMPLEMENTED: Move the Google API key files to Shared**
_/var/www/instance/releases/YYYYMMddHHmmss/config/initializers_

*Copy the deployed from the cloned folder to shared*
```
cp timezone.rb  /var/www/instance/shared/config/initializers/timezone.rb
cp geocoder.rb  /var/www/instance/shared/config/initializers/geocoder.rb
cp staticmap.rb  /var/www/instance/shared/config/initializers/staticmap.rb
```

* Remove the files from the current cloned initializers folder*
```
rm timezone.rb
rm geocoder.rb
rm staticmap.rb

#Go back to app root folder
cd /var/www/instance/releases/YYYYMMddHHmmss
```

* Create the symbolic links to the Google API key files in shared*
```
ln -s /var/www/efw/shared/config/initializers/timezone.rb config/initializers/timezone.rb
ln -s /var/www/efw/shared/config/initializers/geocoder.rb config/initializers/geocoder.rb
ln -s /var/www/efw/shared/config/initializers/staticmap.rb config/initializers/staticmap.rb
```

**8. Create symbolic of the current folder**
_/var/www/instance/_

```
cd /var/www/instance
rm current
ln -s /var/www/efw/releases/YYYYMMddHHmmss/ current

```

**8. Restart server, resque pool and check services**
_/var/www/instance/current_

```
cd current
sudo service httpd restart
passenger-status
#Check resque pool workers
ps aux | grep resque

#Kill the main resque process
kill -9 XXXX

#Restart resque pool workers
bundle exec resque-pool --daemon --environment *environment*

redis-cli ping
service redis status
service httpd status
passenger-status

```

**9. IF NOT IMPLEMENTED: Edit Cronjobs to run under the current folder instead of "dated" deployment folder**
_/var/www/instance/current_

```
#List all cronjobs
crontab -l

#If there are cronjobs pointed to the "dated" deployment folder then, we edit
crontab -e
```

*Replace the dated folder with current folder on vim*
```
%s/releases\/XXdatedFolder/current/gg
```
*After this rerun step 8*

### Performing a pull request from a release folder cloned previously

**1. On local repository create a new feature/hotfix/release branch**
* _on local repositiry root_

```
git flow feature start IM-35
```

**2. Make the necessary changes and commit our changes**
* _on local repositiry root_

```
git add -A
git commit -m "Ticket: Description of changes"
```

**3. Push our feature/hotfix/release branch to GitHub**
* _on local repositiry root_

```
git push -u origin feature/branchName
```
_the terminal will ask for username and password for GitHub_


**4. On production server, create the new branch and pull it from github**
* _on target repository root_

```
git branch feature/branchName
git fetch
git branch --set-upstream-to=origin/feature/branchName feature/branchName
git checkout feature/branchName
git pull
```
_In case any file provokes a failure for being ahead of master, checkout the file and pull again_

**5.1.0. After validations and testing is performed, if changes are valid, close the feature branch on local repository and release it, then push it to GitHub**
* _on local repositiry root_

```
git flow feature finish branchName
git flow release start versionTag
git flow release finish versionTag
gut push -u origin master
```

**5.1.1. On production server, checkout to master branch and perform a pull from github**
* _on target repository root_

```
git checkout master
git pull
```


**5.2. After validations and testing is performed, if changes are not valid, checkout to master on the production server**
* _on local repositiry root_

```
git checkout master
```

**5.2.2. Repeat steps 2 & 3, then checkout to the feature branch perform a pull on the production server. Repeat from step 5.1 or 5.2**
* _on target repository root_

```
git checkout feature/branchName
git pull
```
