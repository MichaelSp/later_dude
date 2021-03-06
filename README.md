LaterDude
=========

[![Build Status](https://secure.travis-ci.org/clemens/later_dude.png)](http://travis-ci.org/clemens/later_dude)

LaterDude is a small calendar helper plugin for Rails with i18n support.

It was heavily inspired by "Jeremy Voorhis'/Geoffrey Grosenbach's calendar_helper plugin":http://topfunky.net/svn/plugins/calendar_helper/. Here's what's different:

* LaterDude leverages the power of the "Rails i18n plugin":http://rails-i18n.org. This means that if you have Rails 2.2 or higher (or if you've installed the i18n gem and the plugin separately), you can just drop in locales and you'll get localized month and day names.
* Where the calendar_helper had one helper method that internally called multiple other private methods, LaterDude keeps the namespace clean by tucking away all of its functionality in a presenter (of sorts).
* Most CSS class names are given implictly, e.g. "day", "weekend", "saturday" (both are given to the displayed month as well as to the previous and following month), "otherMonth" (all days that don't belong to the displayed month) and "today". The only CSS class that can be configured is the table's class (which defaults to "calendar"). Further customization can be taken care of by passing a block (see examples).

Installation
============

You can use LaterDude either as a gem (preferred) or as a Rails plugin.

To use the gem version, put the following gem requirement in your environment.rb:

    config.gem 'later_dude', :source => 'http://gemcutter.org'

Or with Rails 3, add the gem requirement to your Gemfile:

    gem 'later_dude', '>= 0.3.1' # 0.3.0 was broken!

To install it as a plugin, fire up your terminal, go to your Rails app and type:

    $ ruby script/plugin install git://github.com/clemens/later_dude.git

To use the CalendarHelper, put this in a controller:

    helper LaterDude::CalendarHelper

Examples
========

There are two basic ways of outputting a calendar: via the presenter itself or by using the helper method:

    <%= calendar_for(2009, 1) %> and
    <%= LaterDude::Calendar.new(2009, 1).to_html %>

both yield the same results. This is similar to what Rails does with RedCloth/BlueCloth filters.

In addition to year and month which are required parameters you can optionally supply an options hash:

    <%= calendar_for(2009, 1, :calendar_class => "my_calendar", :first_day_of_week => 1)

Possible options are:

* :calendar_class: The CSS class for the surrounding table. Defaults to "calendar".
* :first_day_of_week: The first day of the week (0 = Sunday, 1 = Monday etc.). Defaults to the :'date.first_day_of_week' translation in the respective locale or 0 if no translation can be found.
* :hide_day_names: Hide the table header showing day names. Defaults to false.
* :hide_month_name: Hide the table header showing the month name. Defaults to false.
* :use_full_day_names: Use full instead of abbreviated day names. Defaults to false.
* :use_full_month_names: Use full instead of abbreviated month names. Defaults to true.
* :yield_surrounding_days: Defines whether or not days or the previous and next month are yielded to the passed block. Defaults to false.

* :next_month: Defines if and how the next month should be displayed. See section "Formatting the header section" for further information.
* :previous_month: Defines if and how the previous month should be displayed. See section "Formatting the header section" for further information.
* :next_and_previous_month: Defines if and how the next and previous month should be displayed. See section "Formatting the header section" for further information.
* :current_month: Defines how the current month should be displayed. See section "Formatting the header section" for further information.

You can also pass in a block to mark days according to your own set of rules. The block gets passed each day of the current month (i.e. days of the previous and following month are *not* yielded):

    <%= calendar_for(2009, 1) do |day|
      if Calendar.has_events_on?(day)
        [link_to(day.day, events_path(day.year, day.month, day.day)), { :class => "dayWithEvents" }]
      else
        day.day
      end
    end %>

The block can either return an array containing two elements or a single value. If an array is returned, the first element will be the content of the table cell and the second will be used as HTML options for the table cell tag. If a single value is returned, it will be used as the content of the table cell.

Hint: You can avoid cluttering up your views and move the block to a helper:

    # app/helpers/calendar_helper.rb
    def calendar_events_proc
      lambda do |day|
        if Calendar.has_events_on?(day)
          [link_to(day.day, events_path(day.year, day.month, day.day)), { :class => "dayWithEvents" }]
        else
          day.day
        end
      end
    end

    # app/views/calendar/show.html
    <%= calendar_for(2009, 1, &calendar_events_proc)

Formatting the header section
=============================

You can customize the calendar's header section (i.e. the first table row) by using one or more of the following options:
* :next_month
* :previous_month
* :next_and_previous_month
* :current_month

All of these can either take a single value or an array of two values:

    # simple string
    :next_month => "&raquo;"

    # strftime string
    :next_month => "%b"

    # proc - will evaluate the proc
    :next_month => lambda { |date| link_to "&raquo;", calendar_path(date.year, month.year) }

    # simple string and proc - will evaluate the proc and use it for link_to, e.g. link_to("&raquo;", calendar_path(date.year, month.year))
    :next_month => ["&raquo;", lambda { |date| calendar_path(date.year, month.year) }]

    # strftime string and proc - will evaluate the proc and use it for link_to, e.g. link_to("Jan", calendar_path(date.year, month.year))
    :next_month => ["%b", lambda { |date| calendar_path(date.year, month.year) }]

If you use a proc it gets passed a Date object carrying the respective month, i.e. the proc for :previous_month gets passed a Date object representing the previous month etc.

You can do all kinds of mighty stuff with a proc. Imagine a situation where you want to show the abbreviated month name and then the number of events for the given month:

    # depending on your routes, this outputs something like <a href="/calendar/2009/1">Jan (3 events)</a>
    :next_month => lambda { |date| link_to I18n.localize(date, :format => "%b (#{pluralize(Event.in_month(date))})", calendar_path(date.year, date.month) }

The option :next_and_previous_month can be used as a shortcut if you want to use the same formatting for the next and previous month. Both, :next_month and :previous_month override the :next_and_previous_month if given.

i18n
====

You can define the following keys in your locale files to customize output:

* date.formats.calendar_header: Defines the date format to be used for the table header.
* date.first_day_of_week: Defines the first day of a week (0 = Sunday, 1 = Monday etc.).

Of course, you can also put other configuration options in the locale and then pass them in manually:

    # somewhere in a locale
    date:
      calendar:
        calendar_class: my_cool_calendar

    # in the view
    <%= calendar_for(2009, 1, :calendar_class => I18n.translate(:'date.calendar.calendar_class', :default => "my_calendar"))

Note that you should always pass in a default in case a translation can't be found since your options override the default options which might result in weird exceptions if the translation is missing.

Bugs & Feedback
===============

You can send me feedback, bug reports and patches via "GitHub":http://github.com/clemens.

Copyright (c) 2009-2012 Clemens Kofler <clemens@railway.at>, released under the MIT license.
