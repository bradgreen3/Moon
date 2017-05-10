---
layout: post
title: Cleaning Up Your Views
date: 2016-11-08
excerpt: "Remove logic from your Rails views"
tags: [ruby, rails]
comments: true
---

It’s a good idea to keep your views clean, but what exactly does this mean? In short, aim to keep your views free of logic and use partials to avoid repeating yourself. Partials are particularly useful for forms that’ll be used more than once.
In a job tracking application that I recently created, my views were admittedly a disaster. Below this paragraph you’ll see a view that renders all of the companies and their associated jobs, with the companies sorted by location. You’ll notice that my first loop is calling a method written in the company model. This is where I’ve gone wrong!

{% highlight ruby %}
<h1> Companies with Jobs Sorted By Location </h1>
<% Company.sort_by_city.each do |company| %>
  <p> City: <%= company.city %> </p>
  <p> Company: <%= link_to company.name, company_path(company)%> </p>
    <u> Jobs: </u>
    <% company.jobs.each do |job| %>
    <p> <%= job.title %> </p>
  <% end %>
  <br>
<% end %>
{% endhighlight %}

A more conventional alternative would be to call the sort_by_city method in the company controller, store the result as an instance variable, then iterate over that instance variable in my view.
Within the index method of my company controller, I store the sorted companies as an instance variable called @sorted_companies, then render the view.

{% highlight ruby %}
@sorted_companies = @companies.sort_by_city
  render :location
{% endhighlight %}

With @sorted_companies now available in the view, I no longer need my view to call on the company model. This action is now handled by the controller, thus cleaning up the view and better separating the responsibilities of each component.

{% highlight ruby %}
<% @sorted_companies.each do |company| %>
  <p> City: <%= company.city %> </p>
  <p> Company: <%= link_to company.name, company_path(company)%> </p>
    <u> Jobs: </u>
    <% company.jobs.each do |job| %>
    <p> <%= job.title %> </p>
  <% end %>
  <br>
<% end %>
{% endhighlight %}
