#登录

#批量添加数据
db/seed.rb
```

puts "這個種子檔會自動建立一個admin帳號, 並且創建 10 個 public jobs, 以及10個hidden jobs"
create_account = User.create([email: 'example@gmail.com', password: '12345678', password_confirmation: '12345678', is_admin: 'true'])
puts "Admin account created."
create_jobs = for i in 1..10 do
  Job.create!([title: "Job no.#{i}", description: "這是用種子建立的第 #{i} 個Public工作", wage_upper_bound: rand(50..99)*100, wage_lower_bound: rand(10..49)*100, is_hidden: "false"])
end
puts "10 Public jobs created."
create_jobs = for i in 1..10 do
  Job.create!([title: "Job no.#{i+10}", description: "這是用種子建立的第 #{i+10} 個Hidden工作", wage_upper_bound: rand(50..99)*100, wage_lower_bound: rand(10..49)*100,is_hidden: "true"])
end
puts "10 Hidden jobs created."
```
最后运行命令：
`heroku run rake db:seed`
#清空数据
1. heroku pg:reset DATABASE
他会显示警告，将把所有资料移除，重复打heroku的专案名称即可。
2. heroku run rake db:migrate
建立新的资料库
3. heroku run rake db:seed
跑新的seed