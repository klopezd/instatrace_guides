# instatrace_guides
Guides for change request

## IM-86
1. Insert new setting on DB
```
INSERT INTO settings SET `id` = 8, `name` = 'EnableAbnormalActivityAlertEmail', `value` = '1', `description` = 'Enable Abnormal Activity Alert Email (Turn on: 1: Turn Off: 0)', `created_at` = '2018-02-25 16:51:25', `updated_at` = '2018-02-25 16:51:25';
```

2. Create the new Mailer function on app/mailer.rb

3. Count the milestones inserted for the current shipment. Then add setting validation to Mass_Update workflow and mailer alert.
.⋅⋅* app/controllers/api/shipments_controller.rb near ln 331

```ruby
milestone_count = Milestone.count(:conditions => "shipment_id = #{shipment.id}")
if milestone_count > 10
  setting = Setting.find_by_name('EnableAbnormalActivityAlertEmail')
  if setting && setting.value == "1"
    Mailer.abnormal_activity_notifier(shipment).deliver
  end
end
```
