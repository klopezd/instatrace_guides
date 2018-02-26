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
ADD COLUMN `is_enabled` VARCHAR(255) NULL DEFAULT 'Y' AFTER `is_activated`,
```
2. Add new collection for the new flag value on edit page
* app/models/user.rb near ln 17
```ruby
# IM-86 => Added new input combo for disabled/enabled user - EL - 20180226 .ns
enum enable: { Y: 'Yes', N: 'No' }
# IM-86 => Added new input combo for disabled/enabled user - EL - 20180226 .ne
```

enum enable: { Y: 'Yes', N: 'No' }
2. Add new input combo box on User's edit page
* app/views/active_admin/users/_form.html.erb near ln 20
```ruby
<!-- IM-86 => Added new input combo for disabled/enabled user - EL - 20180226 .ns -->
<%= f.input :is_enabled, :as => :select,:collection => User.enable, :include_blank => false %>
<!-- IM-86 => Added new input combo for disabled/enabled user - EL - 20180226 .ne -->
```
