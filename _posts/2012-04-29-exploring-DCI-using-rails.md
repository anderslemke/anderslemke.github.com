---
layout: post
tags : [dci]
category : Programming
title: Exploring DCI using Rails

---
{% include JB/setup %}

Where am I?

What is DCI?

Why use it?

## An example

### Data

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

### Context

Before we send the offer to the mechanics, though, we need some additional validations.

    class Job < ActiveRecord::Base
      validates_presence_of :description
    end

    class Car < ActiveRecord::Base
      validates_presence_of :registration_number
    end

