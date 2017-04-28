# enum_field

[![Build Status](https://semaphoreci.com/api/v1/igor-galeta/enum_field/branches/master/badge.svg)](https://semaphoreci.com/igor-galeta/enum_field)

Enables Active Record attributes to point to enum like objects, by saving in your database
only an integer ID.

## INSTALL:

    gem 'galetahub-enum_field', require: 'enum_field'

## FEATURES:

* Allows creation of Classes with enum like behaviour.
* Allows any number of members and methods in the enum classes.
* Allows an integer id to be used in your database columns to link to the enum members (user.role_id)
* Enables higher abstraction interaction with +AR+ attributes:
  * <code>user.role = Role.admin</code>
  * <code>if user.role.can_edit?</code>
* Saves in your +AR+ tables, only an integer id pointing to the enumeration member.

## SYNOPSIS:

When in an Active Record class, you have an attribute like role, state or country you have
several options.

* You can create a roles, states or countries table, and dump there all possible values.
* You can use a string to identify, for instance, the role.
* You can use an id to identify the role.

If you are not comfortable with any of this options, maybe +enum_field+ is an answer for you.

## BASIC USAGE:

Define rules:

    class Role
      include EnumField::DefineEnum

      define_enum do
        member :admin
        member :manager
        member :employee
      end
    end

    class User < ActiveRecord::Base
      extend EnumField::EnumeratedAttribute

      # in the database table there is a role_id integer column
      enumerated_attribute :role
    end

Usage:

    user.role = Role.manager
    user.role_id == Role.manager.id  # will be true

    Role.manager.name # :manager
    user.role.name # :manager

    User.first.role.id == User.first.role_id  # will be true

    Role[:manager] == Role.manager # will be true

    instance = Role[:employee]
    instance.admin?    # false
    instance.employee? # true

Your enum classes can have all the methods you need:

    class PhoneType
      include EnumField::DefineEnum

      def initialize(name)
        @name = name
      end

      define_enum do
        member :home,       object: new('home')
        member :commercial, object: new('commercial')
        member :mobile,     object: new('mobile')
      end
    end

    user.phone.type.name

You have some +AR+ like methods in enum classes

    PhoneType.all == [PhoneType.home, PhoneType.commercial, PhoneType.mobile]  # ordered all
    PhoneType.first == PhoneType.home
    PhoneType.last == PhoneType.mobile

    PhoneType.find_by_id(PhoneType.home.id) == PhoneType.home
    PhoneType.find_by_id(123456) == nil
    PhoneType.find(2) == PhoneType.commercial
    PhoneType.find(123456)  # will raise

    PhoneType.find([1, 2]) == [PhoneType.home, PhoneType.commercial]

### Start id from specific number

    class CommentType
      include EnumField::DefineEnum

      define_enum id_start_from: 100 do
        member :video
        member :audio
        member :text
      end
    end

    CommentType.video.id # 101
    CommentType.audio.id # 102
    CommentType.text.id  # 103

## Tests

    bundle install
    bundle exec rspec ./spec/

## LICENSE

(The MIT License)

Copyright (c) 2017 Fodojo LLC
