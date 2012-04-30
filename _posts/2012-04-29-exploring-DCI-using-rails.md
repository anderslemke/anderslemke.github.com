---
layout: post
tags : [dci]
title: Context specific validations when using DCI and Rails
---
{% include JB/setup %}

Where am I?

What is DCI?

Why use it?

Context and context. Which is which?

## An example

### Basic validations

At autobutler.dk we have car owners that has cars, which has jobs. When saving these objects, we only _really_ need the basic relations to be present. Giving us a validation scheme like this:

    class CarOwner < ActiveRecord::Base
    end

    class Car < ActiveRecord::Base
      belongs_to :car_owner
      validates_presence_of :car_owner
    end

    class Job < ActiveRecord::Base
      belongs_to :car
      validates_presence_of :car
    end

### Context validations

Now, let's say we want to make sure, that whenever we send a job to our mechanics, we want to validate the presence of a description.

    class Job < ActiveRecord::Base
      belongs_to :car
      validates_presence_of :car
      validates_presence_of :description, :on => :sending_to_mechanics
    end

This establishes an implicit contract between the context `SendToMechanic` an these validations.

So whenever we manipulate a `Job` in this DCI-context, we will have to specify the validation context.

    class SendingToMechanics
      attr_accessor :job_to_send

      def initialize(job_id)
        @job_to_send = Job.find job_id
        @job_to_send.extend(Sendable)
      end

      def execute
        @job_to_send.send!
      end

      module Sendable
        def send!
          if self.valid?(:sending_to_mechanics)
            self.update_attribute(:verified, true)
          end
        end
      end
    end