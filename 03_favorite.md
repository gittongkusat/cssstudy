作为应聘者对感兴趣的职业未必第一时间进行提交简历，当中包含有相关工作经验或无经验但有意作为发展方向等客观情况，收藏则是起到帮助应聘者对工作的进一步筛选或者发展方向参考的作用。

招聘网站中首先我们对job建立user_id栏目,

>rails g migration add_user_id_to_job

在db文件里找回刚建的xxx_add_user_id_to_job，然后定义change

>def change
add_column :jobs, :user_id, :integer
end

随后建立一个叫favorite的model，资料表有两个栏位：job_id(integer 数字属性)、 user_id ( integer 数字属性）
>rails g model favorite job_id:integer user_id:integer

将资料库建立起来rake db:migrate
>rake db:migrate

建立favorite、job、user三者的关系，跟我们Rails101练习建立群组、文章和用户方式一样
在models下favorite.rb内容中输入
>belongs_to :user
belongs_to :job

job.rb内容中输入

>belongs_to :user
has_many :favorites
has_many :fans, through: :favorites, source: :user

user.rb内容中键入
>has_many :jobs
has_many :favorites
has_many :favorite_jobs, :through => :favorites, :source => :job

创建favorite的controller并输入以下内容
>rails g controller favorite

>def index
@jobs = current_user.favorite_jobs
end

设定config/routes.rb

>resources :jobs do
put :favorite, on: :member
resources :favorite do
end

增加定义jobs_controller.rb

>def favorite
@job = Job.find(params[:id])
type = params[:type]
if type == "favorite"
current_user.favorite_jobs << @job
redirect_to :back
elsif type == "unfavorite"
current_user.favorite_jobs.delete(@job)
redirect_to :back
else
redirect_to :back
end
end

为判定用户是否已收藏对应招聘职位，在models的user.rb中定义
>def is_favorite_of?(job)
favorite_jobs.include?(job)
end

由于我计划在index首页面中直接为每个职位添加该收藏或取消收藏功能，因此在<% @jobs.each do |job| %>循环内加入如下条件

><% if current_user && current_user.is_favorite_of?(job) %>
<%= link_to('取消收藏'.html_safe, favorite_job_path(job, type: "unfavorite"), method: :put) %>
<% else %>
<%= link_to('添加收藏'.html_safe, favorite_job_path(job, type: "favorite"), method: :put) %>
<% end %>

Snip20170128_54.pngaLheNtfRHO5hu1v8rw2n_Snip20170128_54.png800x458 81.8 KB

建立views/favorite/index.html.erb页面，内容可以类似于jobs/index.html.erb形式列出已收藏的职位。
>touch app/views/favorite/index.html.erb

没有登录时点击收藏按钮会出现报错，在job_controller.rb加上“:favorite”，限制点击时需要登录
>before_filter :authenticate_user! , only: [:new, :edit, :create, :update, :destroy, :favorite]

最后在navbar加入已收藏的职位连接，大功告成！

><%= link_to(' 已收藏的工作'.html_safe, favorite_index_path) %>

Snip20170128_56.png
