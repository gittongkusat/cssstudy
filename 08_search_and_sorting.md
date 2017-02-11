步骤：
1. 在gemfile中填入 gem ‘ransack’， bundle install
2. 将app/controllers/jobs_controller.rb 中删除原先index的内容改为：
```

def index
  @search = Job.ransack(params[:q]）
  @jobs = @search.result.where(is_hidden: false)
end
```
3.在html页面中加入搜索框
在views/jobs/index.html.erb 中加入
```
<div class= "search">
  <%= search_form_for @search do |f| %>
    <%= f.label :title_cont %>
    <%= f.search_field :title_cont %>
    <%= f.submit %>
  <% end %>
</div>
```

这时就已经实现可以在工作页面进行搜索了，但是不能排序。

4.增加排序功能：
修改views/jobs/index.html.erb 
```
<div class="dropdown clearfix pull-right">
    <button class="btn btn-default dropdown-toggle" type="button" id="dropdownMenuDivider" data-toggle="dropdown" aria-haspopup="true" aria-expanded="true">
      排序
        <span class="caret"></span>
    </button>
    <ul class="dropdown-menu" aria-labelledby="dropdownMenuDivider">
        <li>
-          <%= link_to("按照薪资下限排序", jobs_path(:order => "by_lower_bound")) %>
+          <%= sort_link @search, :wage_lower_bound, "薪资下限排序" %> 
       </li>
        <li>
-          <%= link_to("按照薪资上限排序", jobs_path(:order => "by_upper_bound")) %>
+          <%= sort_link @search, :wage_upper_bound, "薪资上限排序" %>
        </li>
        <li>
-          <%= link_to("按照发表时间排序", jobs_path ) %>
+          <%= sort_link @search, :created_at, "创建时间顺序" %>
        </li>
    </ul>
</div>
```
好了，这时候的排序如果你点击两下的话是可以顺和逆都可以排的。