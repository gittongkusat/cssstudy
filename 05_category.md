# step1：跟搜索的步骤一样，先装好几个gem
安装这三个gem
- gem 'ransack'
- gem 'will_paginate'
- gem 'seo_helper'
跑bundle install。
#step2：也跟搜索一样，依葫芦画瓢，不过要把方法名和参数名改下(这里把search替换为city，把query_string替换为cuery_string）
修改controller，添加city这个动作和相关参数
```

app/controller/jobs_controller.rb 
+  before_action :validate_city_key, only: [:city]

   ……

+  def city
+    if @cuery_string.present?
+      city_result = Job.published.ransack(@city_criteria).result(:distinct => true)
+      @jobs = city_result.paginate(:page => params[:page], :per_page => 6 )
+    end
+  end
  
  protected
  
+  def validate_city_key
+    @cuery_string = params[:c].gsub(/\\|\'|\/|\?/, "") if params[:c].present?
+    @city_criteria = {place_cont: @cuery_string}
+  end

+  def city_criteria(cuery_string)
+    { :title_cont => cuery_string}
+  end
```
这里@city_criteria = {place_cont: @cuery_string}里面的place_cont是只搜索place这个参数，这里的城市名被我命名为place，如果你命名为city，那就要变为city_cont，如果你命名为address，那就要变为address_cont，以此类推。

step3
修改route，加入city这个方法的路径
config/routes.rb

  resources :jobs do
    collection do
      get :search
+     get :city 
    end
  end

step4
修改导航栏，加入

app/views/common/_navbar.html.erb 
```
<div class="city-change">
	<ul class="nav navbar-nav">
		<li style="font-size: 0.4em; color: white; margin-top: 1.5em;">
			<% if @cuery_string.blank?%>
				全国
			<% else %>
				<%= @cuery_string %>
			<% end %>
		</li>
		<li class="dropdown">
			<a href="#" class="dropdown-toggle" data-toggle="dropdown" style="font-size: 0.5em; margin: auto;" >[切换城市]<span class="caret"></span></a>
			<ul class="dropdown-menu" id="bootstrap-override-1">
				<ul class="list-inline" style="30px;">
					<li><%= link_to("全国", jobs_path, class: "bootstrap-override-2") %></li>
					<li><%= link_to("北京", city_jobs_path(c:"北京"), class: "bootstrap-override-2") %></li>
					<li><%= link_to("上海", city_jobs_path(c:"上海"), class: "bootstrap-override-2") %></li>
					<li><%= link_to("深圳", city_jobs_path(c:"深圳"), class: "bootstrap-override-2") %></li>
					<li><%= link_to("广州", city_jobs_path(c:"广州"), class: "bootstrap-override-2") %></li>
					<li><%= link_to("厦门", city_jobs_path(c:"厦门"), class: "bootstrap-override-2") %></li>
				</ul>
			</ul>
		</li>
	</ul>
</div>
```

step5
增加对应页面，跑touch app/views/jobs/city.html.erb，加入内容其实跟search.html.erb是几乎一样的
app/views/jobs/city.html.erb 
```
<h2 class="text-center"> 位于<%= @cuery_string %>的相关岗位 </h2>
    <% @jobs.each do |job| %>
    <div class="col-md-4 col-xs-12">
    <div class="columns">
      <ul class="text-center panel panel-info panel-pricing list-group">
        <li class="panel-heading"><i class="fa fa-desktop"></i><h3><%= link_to(render_highlight_content(job, @query_string), job_path(job)) %></h3></li>
        <li class="panel-body"><%= job.description %></li>
        <li class="highlight">薪酬范围：￥<%= job.wage_lower_bound %>-<%= job.wage_upper_bound %></li>
        <li class="highlight">创建时间：<%= job.created_at.to_date %></li>
        <li class="panel-footer grey btn btn-default btn-info"><%= link_to("查看详情", job_path(job)) %></li>
      </ul>
    </div>
    </div>
    <% end %>
<div class="text-center col-md-12 pagination">
  <%= will_paginate @jobs, renderer: BootstrapPagination::Rails %>
</div>
<div class="row"></div>
```

