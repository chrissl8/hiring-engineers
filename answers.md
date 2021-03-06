<h1>Datadog Support Engineer Challenge</p>
<p>By Chris Slaight</p>

<h2>Level 1</h2>

<p>To answer the question regarding the Agent, this is essentially the component of the Datadog software ecosystem that resides within whatever host you choose to monitor. Its essential purpose is to collect metrics and events and then send them into the Datadog cloud, where you can truly make something of the data.
<br><br>
After installing the Agent on my local MacBook and grabbing an API key, I was able to successfully create a small Ruby app to send an Event back to Datadog, which also forwarded to my email inbox by the use of tagging myself.
</p>
```ruby
require 'rubygems'
require 'dogapi'
#Define api_key from Datadog
api_key = "6dbff62a24a033178e720f2907618ec6"
#Create dog object
dog = Dogapi::Client.new(api_key)
#Call emit_event to send a new event back to Datadog, tagging my username to generate an email
dog.emit_event(Dogapi::Event.new('Send me an email @chrisjslaight@gmail.com '))
```

<h2>Level 2</h2>

<p>For this level, I've chosen to implement Dogstatsd on my Cloud9 IDE Ubuntu development server for a holiday shopping list Rails app I've been working on. The below method was placed in the main application_controller.rb file to be accessible by all controllers:</p>
```ruby
def render_page
	#This is to be called for any generic web page load
	statsd = Statsd.new
	statsd.increment('web.page_views')
	return 'Page viewed.'
end
```
<p>For the actual implementation, I added a before_action for all page navigation controller actions for my primary controller, people_controller.rb:</p>
```ruby
before_action :render_page, only: [:index,:export,:new,:edit,:show]
```
<h2>Level 3</h2>

<p>For this challenge, I chose to capture metrics on the 'Create Person' database save operation for my shopping app. This was accomplished by modifying the 'create' action of the people_controller.rb file as follows:</p>

```ruby
def create
    #Define start time for Datadog metrics
    startTime = Time.now
    #Define @person instance variable based on explicit parameters
    @person = Person.new(p_params)
    #Set new person's user to the current user
    @person.user = current_user
    #Save the person to the database
    @person.save
    #Define end time for Datadog metrics
    transactionDuration = Time.now - startTime
    #Send to Datadog
    statsd = Statsd.new
    statsd.histogram('support.people.create', transactionDuration)
    #If successful, redirect to this person's view
    if @person.save
      redirect_to @person
    #Otherwise, render the form again (will include future validation)
    else
      render 'new'
    end
  end
```

<p>This allows me to capture and graph these queries in the below Dashboard:</p>

![Image of Datadog Chart]
(http://chrisslaight.com/dev/datadog/Level_3_screenshot.png)

<a href="https://app.datadoghq.com/dash/85692/create-person-metrics">Link to the Dashboard in Datadog</a>

<h2>Level 4</h2>

<p>With this exercise, the three pages chosen to test with would be my ShopWithRails app Home, Login and Register pages. Essentially, the graphs are spiky due to the intermittent nature of the page views being rendered. First, I've implemented the following methods in the main application_controller.rb file:</p>

```ruby
#Method for home page calls
def view_page_home
  statsd = Statsd.new
  statsd.increment('support.page:home')
  return 'Home page viewed.'
end

#Method for login page calls
def view_page_login
  statsd = Statsd.new
  statsd.increment('support.page:login')
  return 'Login page viewed.'
end

#Method for register page calls
def view_page_register
  statsd = Statsd.new
  statsd.increment('support.page:register')
  return 'Register page viewed.'
end
```

<p>Next, the following headers were added to the specific page rendering actions within the welcome_controller (for Home), users_controller (for Register) and logins_controller (for Login): </p>

```ruby
#Call Datadog API on loading of home page
before_action :view_page_home, only: [:index]
```

```ruby
#Call Datadog API on loading of login page
before_action :view_page_login, only: [:new]
```

```ruby
#Call Datadog API on loading of register page
before_action :view_page_register, only: [:new]
```
<p>Finally, I combined the metrics into to the below Dashboard in Datadog:</p>

![Image of Datadog Chart]
(http://chrisslaight.com/dev/datadog/level_4_dashboard.png)

<a href="https://app.datadoghq.com/dash/85603/homeloginregister-page-views">Link to the Dashboard in Datadog</a>

<h2>Level 5</h2>

<p>For the final exercise, I was able to utilize the Datadog Agent running locally on my MacBook to write an Agent Check that samples a random number. To implement this, first I added the following 'random.yaml' file under the conf.d directory:</p>

```yaml
init_config:

instances:
    [{}]
```

<p>Finally, I created a 'random.py' Python file, that actually calls the random number and sends this metric to Datadog:</p>

```python
import random
from checks import AgentCheck
class RandomNumberCheck(AgentCheck):
    def check(self, instance):
        self.gauge('test.support.random', random.random())
```

<p>Screenshots and link to Dashboard:</p>

![Image of Datadog Chart]
(http://chrisslaight.com/dev/datadog/level_5_screenshot.png)

<a href="https://app.datadoghq.com/dash/86562/random-number-from-mbp">Link to the Dashboard in Datadog</a>