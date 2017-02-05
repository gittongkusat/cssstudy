#在navbar上添加搜索栏并实现搜索结果
###STEP1:在navbar中添加views代码
在_navbar.html.erb中的某个位置添加代码如下代码：
>     <div class="nav navbar-nav navbar-left">
      <%= form_tag "#" , :method => :get  do %>
      <div class="input-group" id="search-id" >
        <input type="text" class="form-control" name="q" value="<%= params[:q] %>" placeholder="请输入职位信息">
        <div class="input-group-btn">
          <button class="btn btn-default" type="submit">
            <i class="glyphicon glyphicon-search"></i>
          </button>
        </div>
      </div>
      <% end %>
    </div>

备注：
1. 里面的search-id的css要自己去定义。您也可以参考我的源码：https://fullstack.xinshengdaxue.com/works/32
2. 参考css样式
>'#search-id {
width: 200px;
padding-top: 7px;
}


###STEP2: 安装gems
在Gemfile中加入如下gems:
>gem ‘ransack’
gem ‘will_paginate’
gem 'will_paginate-bootstrap'
gem ‘seo_helper’

备注：
1. gem 'will_paginate-bootstrap'用来美化分页效果的。
2. 执行：bundle install。执行完毕后，重新开启rails server。

###STEP3: 在jobs_controller.rb中添加相关功能方法
在private之上添加如下代码：
>def search
    if @query_string.present?
      search_result = Job.ransack(@search_criteria).result(:distinct => true)
      @jobs = search_result.paginate(:page => params[:page], :per_page => 5 )
    end
  end

  >protected

  >def validate_search_key
    @query_string = params[:q].gsub(/\\|\'|\/|\?/, "") if params[:q].present?
    @search_criteria = search_criteria(@query_string)
  end


  >def search_criteria(query_string)
    { :title_cont => query_string }
  end

备注：
把上面

>search_result = Job.ransack(@search_criteria).result(:distinct => true)

改成：
>search_result = Job.published.ransack(@search_criteria).result(:distinct => true)

就会只搜索发表出来的职位，而不是所有职位。
接着在最开头添加：
>before_action :validate_search_key, only: [:search]

###STEP4: 修改routing
在routes.rb中把
>resources :jobs do
 resources :resumes
end

修改成：
>resources :jobs do
  resources :resumes
  collection do
   get :search
   end
  end

重新开启rails server。

###STEP5: 新建搜索页面
touch app/views/jobs/search.html.erb
在里面添加如下内容：
><div>
  <br>
  <% if @jobs.blank? %>
    <br>
    <'h2 class="search-info-title">搜索信息不能为空，请输入关键字搜索</h2>
  <% else %>
    <'h2 class="search-info-title"> 有关"<%= @query_string %>"的工作信息 </h2>
    <div class="search-result">
      <div class="row jobs-show0"></div>
      <div class="job-table">
        <% @jobs.each do |job| %>
        <div class="row jobs-show">
          <div class="col-md-12 col-lg-9 col-lg-offset-1">
            <div class="pull-right">发布时间：<%= job.created_at.to_date %>
            </div>
            <p ><%= link_to(render_highlight_content(job,@query_string),job_path(job)) %></p>
          </div>
        </div>
        <% end %>
      </div>
    </div>
    <div class="text-center">
      <%= will_paginate(@jobs, renderer: BootstrapPagination::Rails) %>
    </div>
  <% end %>
</div>

备注：
里面的search-info-title，jobs-show0等 的css要自己去定义。也可以参考源码：https://fullstack.xinshengdaxue.com/works/32
###STEP6: 修改navbar内容
把Step 1中的 
><%= form_tag "#" , :method => :get do %>

修改成：
  ><%= form_tag search_jobs_path , :method => :get do %> 

###STEP 7: 实现搜索highlight功能
在app/helpers/jobs_helper.rb里添加如下代码
>def render_highlight_content(job,query_string)
  excerpt_cont = excerpt(job.title, query_string, radius: 500)
  highlight(excerpt_cont, query_string)
  end

接着把app/views/jobs/search.html.erb中的如下代码做修改：
><%= link_to(job.title,job_path(job)) %>

改成：
><%= link_to(render_highlight_content(job, @query_string), job_path(job)) %>


###补充：
1.from Daniel
>我提交了修改方法到你的 pull requests 你可以参考一下：
https://github.com/RichardWeiYang/job-listing/pull/1/commits/23943b5107f846508d9d0cb725b8727e6d4ae71e
> 
另外大多数同学的搜索框靠左边，下面提供了搜索框靠右并且居于「用户登录」下拉菜单之前的解法：
https://github.com/RichardWeiYang/job-listing/pull/2/commits/5df93d881963d62751ca809ad8dadfef67a1a3b7


2.在复制代码时请将上文中h2/#前的'去掉，这个markdown会直接将h2/#解读把代码中的汉字放大～
###版权声明
[原文链接](http://forum.qzy.camp/t/navbar/486])
