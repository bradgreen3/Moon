---
layout: post
title: Adventures In Open Source
date: 2017-02-26
excerpt: "My first experience making an open source contribution"
tags: [ruby, rails, open source, factory girl]
comments: true
---

I long held the perception that open source projects were out of my league, so I stayed away from them. I recently decided to challenge this belief, and to my surprise, I was wrong. There are tons of resources out there to help you get started with open source, and many projects are very friendly to newcomers.

After digging around on sites like codetriage and issuehub, I came up with three solid options for a first open source contribution. Of the three, I decided on a project called Thanksgiving, a simple app that helps users keep track of philanthropic efforts. I was intrigued by its relatively small code base, small numbers of contributors, relatively recent issues, Waffle board, and good setup documentation

### Getting Started
With my project picked and confidence high, I cloned the repo. I opened my terminal and moved into the directory only to receive a notification that Rails isn’t installed on my system. Hmm…I’m pretty sure I’ve built some Rails projects before, but fine terminal, you win. I reinstalled Rails and was feeling confident again. I typed rails s and….“Ruby version is 2.0, but your Gemfile specified 2.2.2.” Ugh! Fortunately, this was a simple fix thanks to RVM. Onward and upward!

I visited localhost:3000 and was greeted with a basic welcome screen. I thought to myself, “what a relief, it’s working”! I noticed the log in was through Google’s oAuth and decided to give it a whirl. What happened next should come as no surprise: another error! This time it was a Faraday SSL issue, so back to Google I went in search of a solution! This was also a relatively easy fix. For whatever reason, reinstalling Ruby did the trick (rvm reinstall 2.2.2 — disable-binary). With a fresh install of Ruby, I finally got logged in through the oAuth process.

### Task 1: Confirm or Refute a Bug
I thought a good introduction to the code base would be trying to resolve a bug. The author of the project created a bug report for not being able to visit the ‘new donation’ page. I opened the console, set my account to approved and admin status, then went to the page in my browser. It worked fine for me. I responded to the issue on Github and asked for more information to help me reproduce it. Ok, that wasn’t so bad. Addressing a bug report is an easy way to gain some momentum, and it’s where I recommend you start.

### Task 2: Fix the Date Field
The most approachable issue I could find was a request to clean up the date input on the ‘new donation’ page and have it default to today’s date.

<figure>
	<a href="/assets/img/ugh.png"><img src="/assets/img/ugh.png"></a>
	<figcaption><a href="/assets/img/ugh.png" title="before">What it looked like before I intervened</a></figcaption>
</figure>

I agree, the date input is kind of ugly and not very user friendly. They were using foundation-datepicker.js, something I’ve never heard of (apparently for good reason). Instead of adding any new dependencies, I switched to the default Rails helper and liked what I saw much better. Easy fix! Setting the default date to the current date was also very easy. Instead of messing around with JavaScript to manipulate the DOM, I just set the default date in the donation controller like so:

{% highlight ruby %}
def new
  @donation = Donation.new(date: Date.today)
end
{% endhighlight %}

And the result:

<figure>
	<a href="/assets/img/yay.png"><img src="/assets/img/yay.png"></a>
	<figcaption><a href="/assets/img/yay.png" title="before">What it looked like after I intervened</a></figcaption>
</figure>

I was feeling good about making a tangible difference, so it was time to submit a pull request. Hot tip: fork the repo first. I tried to push a branch to the main repo, and that doesn’t work unless you have collaborator status. The proper sequence is to fork the repo, clone your fork, commit changes to a branch, then submit a pull request from your branch to the main repo. Bonus points if you reference the issue in your pull request (e.g. Closes #14).

### Task 3: Add Factory Girl
When I saw this issue, I figured it’d be a piece of cake. I’ve used Factory Girl in many of my projects, and it’s not too difficult to set up, especially when consulting fellow Turing student Ryan Flach’s Rails setup Gist. I checked out a new branch, added all of the necessary code for Factory Girl, and once again felt accomplished. I ran the test suite, and….16 failures.

Woah! I switched back to the master branch to make sure this wasn’t my fault. Thankfully, the failures were happening there too. My first mission was fixing as many failures as I could. I got the test suite down to about 6 failures, all of which were occurring in the same file:

{% highlight ruby %}
feature "User management" do
  let(:users_index_path) { polymorphic_path([:users]) }
context "when an admin user visits the users index" do
    let!(:admin_user) { login_new_admin_user }
    subject { admin_user }
before { visit users_index_path }
it "displays a table of users" do
      expect(page).to have_table("users_table")
    end
it "fills the table with the right headers and rows" do
      check_headers_and_values_on_generic_index_page(
        "users_table", {
          "Name" => "Example User",
          "Created At" => subject.created_at_string,
          "Approval At" => subject.approval_at_string,
          "Admin" => "true",
          "Actions" => "Remove Approval Delete",
        }
      )
    end
context "and a new non-admin user is added" do
      subject! { User.create!(name: "Joe") }
before { visit users_index_path }
it "fills the table with that user\"s information too" do
        check_headers_and_values_on_generic_index_page(
          "users_table", {
            "Name" => "Joe",
            "Created At" => subject.created_at_string,
            "Approval At" => "",
            "Admin" => "",
            "Actions" => "Approve Delete",
          }
        )
      end
context "and the current user clicks the approve link for that user" do
        before do
          within(:row_for, subject) do
            click_link "Approve"
          end
        end
it "approves the user" do
          expect(subject.reload).to be_approved
        end
it "displays an approval time" do
          within(:row_for, subject) do
            expect(find(:value_under_header, "Approval At").text).to_not be_blank
          end
        end
it "no longer shows the approve link" do
          within(:row_for, subject) do
            expect(find(:value_under_header, "Actions").text).to eq "Remove Approval Delete"
          end
        end
context "and the current user clicks the Remove Approval link for that user" do
          before do
            within(:row_for, subject) do
              click_link "Remove Approval"
            end
          end
it "removes the user's approval" do
            expect(subject.reload).to_not be_approved
          end
        end
      end
context "and the current user clicks the delete link for that user" do
        before do
          within(:row_for, subject) do
            click_link "Delete"
          end
        end
it "deletes the user" do
          expect(User.find_by_id(subject.id)).to be_nil
        end
      end
    end
  end
end
{% endhighlight %}

This test file overwhelmed me a bit at first. It’s not how I write tests, but that’s OK. A great thing about open source is being exposed to different ideas. I ended up learning a lot from this single file. For example, they use a lot of helper methods. I like the idea, even though here it adds some unnecessary complexity. Also, every test for the user index page is contained in the same file. I split my tests into files based on functionality, but I don’t mind this approach. It just required dealing with some nested context statements that I wasn’t too familiar with. In any case, an interesting thing to learn about.

Because all of the failing tests were in this file, I decided to redo the whole page using Factory Girl. My implementation reduced this file from 92 lines to 74 lines. It also reduced dependency on the terribly-confusing check_headers_and_values_on_generic_index_page helper method. My implementation tests all of the same functionality, but I believe it is much easier to read, all while largely conforming to overall style of the code in the project. Pull request submitted!

{% highlight ruby %}

feature "user management" do
  let(:users_index_path) { polymorphic_path([:users]) }
context "when an admin visits the users index" do
    let!(:admin) { login_new_admin_user }
    let!(:users) { create_list(:user, 2, approval_at: "01/01/2001") }
    let!(:unapproved_user) { create(:user) }
before { visit users_index_path }
it "displays a table of users" do
      expect(page).to have_table("users_table")
    end
it "displays a table with proper headers and admin row data" do
      page.find("#user_#{admin.id} td:nth-of-type(1)", text: admin.name)
      page.find("#user_#{admin.id} td:nth-of-type(2)", text: admin.created_at_string)
      page.find("#user_#{admin.id} td:nth-of-type(3)", text: "Yes")
      page.find("#user_#{admin.id} td:nth-of-type(4)", text: admin.approval_at_string)
      page.find("#user_#{admin.id} td:nth-of-type(5)", text: "Yes")
      page.find("#user_#{admin.id} td:nth-of-type(6)", text: "Delete")
    end
it "displays a table with proper headers and user row data" do
      page.find("#user_#{users.first.id} td:nth-of-type(1)", text: users.first.name)
      page.find("#user_#{users.first.id} td:nth-of-type(2)", text: users.first.created_at_string)
      page.find("#user_#{users.first.id} td:nth-of-type(3)", text: "Yes")
      page.find("#user_#{users.first.id} td:nth-of-type(4)", text: users.first.approval_at_string)
      page.find("#user_#{users.first.id} td:nth-of-type(5)", text: "No")
      page.find("#user_#{users.first.id} td:nth-of-type(6)", text: "Delete")
    end
context "and admin clicks No for user approval" do
      before do
        find('.not-approved').click
end
it "approves the user" do
        expect(unapproved_user.reload).to be_approved
      end
it "displays an approval time" do
        expect(page).to have_content(unapproved_user.approval_at_string)
      end
it "shows Yes instead of No button" do
        expect(page).to have_link('Yes', href: remove_approval_user_path(unapproved_user))
      end
context "and the admin clicks No for user approval for same user" do
        before do
          find(:xpath, "//a[@href='/users/#{unapproved_user.id}/remove_approval']").click
        end
it "removes the user's approval" do
          expect(unapproved_user.reload).to_not be_approved
        end
      end
context "and the admin clicks the delete link for same user" do
        before do
          within(:css, "#user_#{unapproved_user.id}") do
            click_link "Delete"
          end
        end
it "deletes the user" do
          expect(User.find_by_id(unapproved_user.id)).to be_nil
        end
      end
    end
  end
end

{% endhighlight %}

### Conclusion

I consider my first foray into the land of open source a success. Did I change the world? Probably not, but maybe? Did I learn something? Absolutely. Did I make a positive influence on the project? I think so. If you’re on the fence about contributing to an open source project, I say “go for it”. Find the most approachable project that you can. Look for a small number of contributors, open issues that sound doable, and active users with a history of providing feedback. If you can’t find anything, then let me recommend a cool project that helps users keep track of philanthropic efforts. It’s called <a href="https://github.com/lbraun/thanksgiving">Thanksgiving</a>. Happy programming!
