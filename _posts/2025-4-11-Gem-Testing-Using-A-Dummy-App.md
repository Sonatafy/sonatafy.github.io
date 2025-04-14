---
layout: post
title: Setting Up a Dummy Rails App for Robust Gem Testing
author: Garth Gaughan
---

When developing a Ruby gem that interacts with a Rails application, a dummy app is nice for thorough testing. It's a miniature Rails application within your gem's test suite, providing a realistic environment to test your gem's features.

### Why Use a Dummy App?

A dummy app is used to simulate real-world usage, allowing you to test how your gem behaves within a functioning Rails application. It also enables you to verify that your gem creates the correct files and content, ensures compatibility with core Rails functionalities and other gems, and allows you to run Rails-specific tests, utilizing testing frameworks like RSpec and Capybara.

### Step-by-Step Guide

Add rails as a Development Dependency:
```
group :development, :test do
  gem 'rails', '~> 7.0' # Or your desired Rails version
  # ... other development/test dependencies
end
```

Run `bundle install`.

Create a test/dummy Directory:
```
test/
└── dummy/
```
Generate the Dummy Rails App:

`cd test`
`rails new dummy --skip-bundle --minimal`

--skip-bundle:  Don't run bundle install in the dummy app.

--minimal: Create a basic Rails application.

Configure the Dummy App's Gemfile:
```
source 'https://rubygems.org'
gem 'rails'
gem 'sqlite3' # Or your preferred database
# Include your gem
gem '<your_gem_name>', path: '../../'
# Include other necessary gems
gem 'view_component'
gem 'tailwindcss-rails'
gem 'lookbook'
gem 'rspec-rails'
gem 'capybara'
```

Run `bundle install` in the test/dummy directory.

Configure the Test Environment (rails_helper.rb):
```
# frozen_string_literal: true
require 'spec_helper'
ENV['RAILS_ENV'] ||= 'test'
require_relative './dummy/config/environment'
# Require testing libraries
require 'view_component/test_helpers'
require 'capybara/rspec'
require 'rails/generators/test_case'

RSpec.configure do |config|
  config.include ViewComponent::TestHelpers, type: :component
  config.include Capybara::RSpecMatchers, type: :component
  config.include Rails::Generators::Testing::Behavior, type: :generator
  config.include Rails::Generators::Testing::SetupAndTeardown, type: :generator
  config.include Rails::Generators::Testing::Assertions, type: :generator
  
  config.before(:all, type: :generator) do
    destination File.expand_path('../dummy', __dir__)
    prepare_destination
  end
  # Add other RSpec configurations
  config.use_transactional_fixtures = true
  config.infer_spec_type_from_file_location!
  config.filter_rails_from_backtrace!
end
```

Writing Your Tests:

Generator Tests: Verify file creation.
```
require 'rails_helper'
RSpec.describe MyGem::Generators::MyGenerator, type: :generator do
  # ...
  it 'creates the expected files' do
    # ...
    assert_file 'app/models/my_resource.rb' do |content|
      expect(content).to match(/class MyResource < ApplicationRecord/)
    end
  end
end
```

Component/Integration Tests: Test Rails interaction.
```
require 'rails_helper'
RSpec.describe MyComponent, type: :component do
  it 'renders correctly' do
    render_inline(MyComponent.new(name: 'World'))
    expect(rendered_content).to have_text('Hello, World!')
  end
end
```
Running Your Tests:

`bundle exec rspec`

### Key Considerations

Keep it minimal: Include only necessary dependencies.

Database setup: Configure config/database.yml and run migrations.

Configuration: Set up required Rails configurations.

Teardown: Clean up generated files/data in tests.

### Conclusion

Setting up a dummy Rails application is a crucial step in developing robust and reliable Rails-integrated gems.  This approach provides a controlled environment to simulate real-world usage, effectively test code generation, and verify seamless integration with Rails features.  By following these guidelines, you can ensure your gems, such as my ViewComponentGenerator gem, function correctly and deliver a smooth experience for Rails developers.

