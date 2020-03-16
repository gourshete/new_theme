---
layout: post
title:  "Send Email from Rails Backend"
categories: [ Jekyll ]
image: assets/images/send-email-1.jpg
---

Sending email is integral part of every web application nowadays. In this post we will cover how to send emails from 
a rails application.

First you need an active smtp server to send an email, because only SMTP servers are capable of sending or receiving 
emails. There are many providers such as gmail, sendgrid and more. For this post, we will consider using gmail.

Assuming you have a Rails application with User model.

1. We need to configure SMTP server. This can be set in config/environments/development.rb like
    ```ruby
      config.action_mailer.delivery_method = :smtp
      host = 'example.com' #replace with your own url
      config.action_mailer.default_url_options = { host: host }
      
      # SMTP settings for gmail
      config.action_mailer.smtp_settings = {
          :address              => "smtp.gmail.com",
          :port                 => 587,
          :user_name            => ENV['EMAIL'],
          :password             => ENV['PASSWORD'],
          :authentication       => "plain",
          :enable_starttls_auto => true
      }
    ```
If you are using ZSH, simply add these lines in `.zshrc` file <br>
export EMAIL='example@gmail.com'; <br>
export PASSWORD='PASSWORD';


2. Let's write the code now. What to send in the email? Let's say we want to send a welcome mail when user signs up. We will generate a mailer named 
    ```ruby
   rails g mailer UserRegistration
    ```
   This will generate -
   ![screenshot]({{ site.baseurl }}/assets/images/SS-send-email-1.png)
   
3. Open `app/mailers/user_registrations_mailer.rb` and add following method
    ```ruby
    def registration(user)
        @user = user
        mail(to: user.email, subject: 'Welcome to Rails learning')
    end
    ```

4. Open `app/views/user_registrations_mailer` & add a new .erb file whose name resembles with method name in 
   `app/mailers/user_registrations_mailer.rb`. In our case it is `registration.html.erb`. Now put in whatever welcome message 
   you want a user to see in email. I will simply add
    ```ruby
    Hello <%= @user.first_name%> <%= @user.last_name %>,
    
    Welcome to Ruby on Rails learning.
    
    Thanks for signing up.
    ```
5. We need to trigger the action of sending emails. There are multiple ways to achieve this. Can call this from controller action or a service call or active-record callback. Will prefer last option here.
   In `User` model register `after_create` callback
    ```ruby
    after_create :send_sign_up_email
    .
    .
    .
    private
    def send_sign_up_email
      UserRegistrationsMailer.registration(self).deliver_now
    end
    ```

And you are all set to send sign-up emails - right after user is created.

<br>

- Note -

Sending emails from google account may require allowing access to less secure apps. This setting is set to OFF by default
 for security reasons. To turn it ON,  goto https://myaccount.google.com/lesssecureapps and turn ON 'Allow less secure apps'


<br>

**References** 

- Rails ActionMailer - https://guides.rubyonrails.org/action_mailer_basics.html <br>

- Google less secure apps -https://support.google.com/accounts/answer/6010255?hl=en