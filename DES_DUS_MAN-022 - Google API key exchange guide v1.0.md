## Google API key exchange guide

### Updating to a new API key

**1. Navigate to the shared's initializers**
* _/var/www/instance/shared/config/initializers_

```
cd /var/www/instance/shared/config/initializers
```

**2. Edit the geocoder.rb/timezone.rb/staticmap.rb file**
* _/var/www/instance/shared/config/initializers/_

```
vim geocoder.rb
```

**3. Comment previous API key, don't delete it. Add the new one with the comment below and save changes**
* _/var/www/instance/shared/config/initializers/geocoder.rb_

* If any of the files exists on the releases/YYYYMMddHHss folder must be deleted first and then created (linked) to the shared folder content
 
```ruby
API_KEY_VAR = "Api-key-value" #API Key changed for geocoder/timezone/staticmap on dd Month year by User
```

**4. Restart services, resque and Passenger**
* _/var/www/instance/current_

```
#Restart Apache
sudo service httpd restart

#Check resque pool workers
ps aux | grep resque

#Kill the main resque process
kill -9 XXXX

#Restart resque pool workers
bundle exec resque-pool --daemon --environment *environment*

#Restart passenger
passenger-config restart-app

#Check services status
redis-cli ping
service redis status
service httpd status
passenger-status
```

**5. Repeat on the second production server steps 1-4**
* _/var/www/instance/shared/config/initializers/_
