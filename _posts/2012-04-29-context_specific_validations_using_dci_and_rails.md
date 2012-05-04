---
layout: post
tags : [dci, ruby, rails, architecture]
title: Context specific validations when using DCI and Rails
---
{% include JB/setup %}

_Please note, that the proposed patterns only works **after** [this commit](https://github.com/rails/rails/commit/9b15e01c219013825275e7e10140d9883d148c8c) in Ruby on Rails._

When I first heard about [DCI](http://en.wikipedia.org/wiki/Data,_Context,_and_Interaction), I almost couldn't get my arms down. I have long felt that my models tended to get really fat. Of course, that was a good thing in the "skinny controllers - fat models"-days, but as applications grew, and each model played more and more different _roles_ in more and more different _contexts_, the fat models simply grew too fat.

So we started playing around with DCI on autobutler.dk, which in turn made me think about Rails model validations.

Validations could very well be context specific, and the model itself, i.e. the DCI data, should only care about the validations needed to persist the objects in the database.

## An example

I'll use a real world example, from my current work, autobutler.dk. At autobutler.dk car owners can specify what they need to get done on their car, and get offers from mechanics on that specific job.

### Basic validations

The data model, with basic validations, looks something like this.

    class CarOwner < ActiveRecord::Base ; end

    class Car < ActiveRecord::Base
      belongs_to :car_owner, :validate => true
      validates_presence_of :car_owner
    end

    class Job < ActiveRecord::Base
      belongs_to :car, :validate => true
      validates_presence_of :car
    end

**These are the validations we need for the objects to exist in our database.**

### Context specific validations

Now, let's say we want to make sure, that before we actually show a job to our mechanics, we want to validate the presence of 

* a description on the job,
* a registration number on the car, and
* a phone number on the car_owner.

This could be solved in the DCI context like this:

    # app/contexts/request_for_tender.rb

    class RequestForTender
      attr_accessor :job

      def initialize(job)
        @job = job
        @job.extend(Sender)
      end

      def execute
        @job.send
      end

      module Sender
        def send
          if self.valid?(:request_for_tender)
            self.update_attribute(:verified, true)
          end
        end
      end
    end

    Job.class_eval do
      validates_presence_of :description, :on => :request_for_tender
    end
    Car.class_eval do
      validates_presence_of :registration_number, :on => :request_for_tender
    end
    CarOwner.class_eval do
      validates_presence_of :phone_number, :on => :request_for_tender
    end

What happens here is that

1. we define the validation context `:request_for_tender` on all the models, and
1. whenever we manipulate a Job in this DCI context, we do it in the validation context `:request_for_tender`.

### In the controller

    class JobsController < ApplicationController
      def request_for_tender
        if RequestForTender.new(current_user.jobs.find(params[:id])).execute
          # Everything is OK. Redirect to where it makes sense
        else
          # The job, car, or car_owner is not valid. Show the errors.
        end
      end
    end

## What we gain

Whenever we, as programmers, want to know exactly how a job is set to tender and what it requires, there is only _one place to look_.

## Things to consider

* Should we pass ID's or objects to the context? Here I propose passing a object, which moves the responsibility of verifying that the job can in fact be handled by the current user, to the controller.
* Where should we put the DCI context specific validations. I proposed to put them in the DCI context file to keep it all together. But this might not make sense in all scenarios.
* This only works _after_ [this commit](https://github.com/rails/rails/commit/9b15e01c219013825275e7e10140d9883d148c8c) in Ruby on Rails.
* Should we use exceptions to communicate to the controller that something went wrong as talked about in [this comment](http://mikepackdev.com/blog_posts/24-the-right-way-to-code-dci-in-ruby#comment-454457934)?
* Will we have to go [all the way](https://github.com/cjbottaro/schizo) or is this sufficient?