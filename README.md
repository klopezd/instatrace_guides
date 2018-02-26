# instatrace_guides
Guides for change request

## IM-86

### Runaway activity alert - EFM
1. Insert new setting on DB
```SQL
INSERT INTO settings SET `id` = 8, `name` = 'EnableAbnormalActivityAlertEmail', `value` = '1', `description` = 'Enable Abnormal Activity Alert Email (Turn on: 1: Turn Off: 0)', `created_at` = '2018-02-25 16:51:25', `updated_at` = '2018-02-25 16:51:25';
```

2. Create the new Mailer function 
* app/mailers/mailer.rb at bottom
```ruby
# IM-86 => Added mailer to send Email alert for Abnormal Activity - EL - 20180226 .ns
def abnormal_activity_notifier(shipment)
    @shipment = shipment
    subject = "Abnormal Activity detected for Shipment #{shipment.srf}" 
    mail(:to => MONITORING_ALERT_EMAIL, :subject => subject)
  end
# IM-86 => Added mailer to send Email alert for Abnormal Activity - EL - 20180226 .ne
```

3. Create the mail body text, a new file should be created on app/views/mailer/abnormal_activity_notifier.text.erb

```
The shipment with SRF, <%= @shipment.srf %> has reported abnormal activity on EFM

There are over 10 milestones registered for this shipment. Please review

Thank you and have a great day!

----
regards,
EFM team

```

4. Count the milestones inserted for the current shipment. Then add setting validation to Mass_Update workflow and mailer alert.
* app/controllers/api/shipments_controller.rb near ln 331

```ruby
# IM-86 => Added count and setting validations to send Email alert for Abnormal Activity - EL - 20180226 .ns
milestone_count = Milestone.count(:conditions => "shipment_id = #{shipment.id}")
if milestone_count > 10
  setting = Setting.find_by_name('EnableAbnormalActivityAlertEmail')
  if setting && setting.value == "1"
    Mailer.abnormal_activity_notifier(shipment).deliver
  end
end
# IM-86 => Added count and setting validations to send Email alert for Abnormal Activity - EL - 20180226 .ne
```
### Deactivate user function - EFM/EFW/TPK
1. Insert new column on the User's table
```SQL
ALTER TABLE `instatrace`.`users` 
ADD COLUMN `is_enabled` VARCHAR(255) NULL DEFAULT 'true' AFTER `is_activated`,
```
2. Add new param ":is_enabled" to the permit_params line
* app/admin/user.rb near ln 22
```ruby
# IM-86 => Added new param for disabled/enabled user - EL - 20180226 .ns
permit_params :first_name, :last_name, :email, :address, :phone, :login, :activation_code, :password, :password_confirmation, :language, :role_id, :is_enabled
# IM-86 => Added new param for disabled/enabled user - EL - 20180226 .ne
```

3. Add new input combo box on User's edit page
* app/views/active_admin/users/_form.html.erb near ln 20
```html
<!-- IM-86 => Added new input combo for disabled/enabled user - EL - 20180226 .ns -->
<%= f.input :is_enabled, :as => :select,:collection => User.enable, :include_blank => false %>
<!-- IM-86 => Added new input combo for disabled/enabled user - EL - 20180226 .ne -->
```
4. Add validations on get_session and activation functions to avoid disabled users to use the token
* app/controllers/api/api_controller.rb inside def activation replace:
```ruby
# IM-86 => Added new token validation for disabled/enabled user - EL - 20180226 .ns
if user
  if user.is_enabled == 'true'
    user.generate_token! if user.id != WORDTRAK_POST_SHIPMENT_USER_ID
    render :json => {:token => user.api_token}.to_json
  else
    render :status => 402, :json => {:errors => "Invalid activation code"}.to_json and return
  end
else
  render :status => 402, :json => {:errors => "Invalid activation code"}.to_json and return
end
# IM-86 => Added new token validation for disabled/enabled user - EL - 20180226 .ne
```

* app/controllers/api/api_controller.rb inside def get_session replace:
```ruby
# IM-86 => Added new token validation for disabled/enabled user - EL - 20180226 .ns
if @user
  if @user.is_enabled == 'true'
    sign_in :user, @user
  else
    render :status => 401, :json => { :errors => "Invalid token" } and return
  end
else
  render :status => 401, :json => { :errors => "Invalid token" } and return
end
# IM-86 => Added new token validation for disabled/enabled user - EL - 20180226 .ne
```
5. Add validations on get_shipment function to avoid disabled users to use the token and get shipment details
* app/controllers/api/shipment_controller.rb replace def get_shipment
```ruby
# IM-86 => Added new token validation for disabled/enabled user - EL - 20180226 .ns
def get_shipment
  @user = User.find_by_api_token params[:token] unless params[:token].blank?
  if @user.is_enabled == 'true'
     mobile_version = params[:mobile_ver] unless params[:mobile_ver].blank?
     if !check_latest_mobile(mobile_version) or params[:mobile_ver].blank?
        render :status => 406, :json => {:errors => "Need upgraded"}.to_json and return
     end

     shipment_id = params[:shipment_id]
     render :status => 403, :json => {:errors => "Incorect shipment number"}.to_json and return if shipment_id == 0
     @shipment = Shipment.api_search(shipment_id)
     render :status => 404, :json => {:errors => "Shipment not found"}.to_json and return unless @shipment
  else
     render :status => 403, :json => {:errors => "Incorect shipment number"}.to_json and return
  end
end
# IM-86 => Added new token validation for disabled/enabled user - EL - 20180226 .ne
```
