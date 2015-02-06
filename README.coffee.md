A rule target Knockout Widget for CCNQ(4)
-----------------------------------------

A rule target is an item in the `gwlist` field of a `rule` entry in a CCNQ4 `ruleset` database. Such a database is used by the [`tough-rate`](https://github.com/shimaore/tough-rate) LCR engine, especially its `routes-registrant`, `routes-carrierid`, `routes-gwid` middleware modules.

This module adds a `rule-target` component in Knockout.

Usage
-----

```coffeescript
{RuleTarget,rule_target} = (require 'ccnq-ko-rule-target') knockout
```

Parameters:
- value: the `gwlist` item
- $root.gateways: a list of valid gateways
- $root.carriers: a list of valid carriers

    module.exports = (ko) ->

      tag_name = 'rule-target'

      rule_target = (f) ->
        {tag} = teacup
        tag tag_name, params: "value:#{f},$root:$root"

      class RuleTarget
        constructor: (data) ->
          assert data?, 'data is required'

Data
----

A `gwlist` item typically contains one of:
- `source_registrant` -- if the rule should route through the caller's registrant;
- `gwid` -- if the call is to be routed out through a given gateway;
- `carrierid` -- if the call is to be routed out through a carrier (a set of gateways with similar costs).

          @source_registrant = ko.observable data.source_registrant
          @gwid = ko.observable data.gwid
          @carrierid = ko.observable data.carrierid
          @_validated = ko.observable false
          return

View Model
----------

We expect `params="value:$data,$root:$root"`. This means `value` is a `RuleTarget` object.

      view = ({value,$root}) ->
        assert value instanceof RuleTarget, 'value should be an instance of RuleTarget'
        {gateways,carriers} = $root
        assert gateways?, 'gateways is required'
        assert carriers?, 'carriers is required'

        chosen = if value.source_registrant() is true
            'registrant'
          else if value.carrierid()?
            'carrier'
          else if value.gwid()?
            'gateway'
        @chosen = ko.observable chosen

        @gwid = value.gwid
        @carrierid = value.carrierid

        gateway_valid = (id) -> id? and id in gateways
        carrier_valid = (id) -> id? and id in carriers

        @valid = ko.computed =>
          switch @chosen()
            when 'registrant'
              @gwid null
              @carrierid null
              true
            when 'gateway'
              @carrierid null
              gateway_valid @gwid()
            when 'carrier'
              @gwid null
              carrier_valid @carrierid()
            else
              @gwid null
              @carrierid null
              false

Flow the data back to the model.

        @chosen.subscribe =>
          value.source_registrant @chosen() is 'registrant'
        @valid.subscribe (is_valid) =>
          value._validated is_valid

        return

HTML
----

      html = ->
        name = "rule-target-#{Math.random()}"
        {a,ul,li,label,input,text} = teacup
        ul '.target', ->
          li '.choice', ->
            label ->
              input
                type:'radio'
                name:name
                value:'registrant'
                required:true
                bind:
                  checked: 'chosen'
              text 'Use Registrant'
          li '.choice', ->
            label ->
              input
                type:'radio'
                name:name
                value:'carrier'
                bind:
                  checked: 'chosen'
                required:true
              text 'Use Carrier '
            input
              list:'carrier'
              name: 'carrierid'
              bind:
                value: 'carrierid'
                enable: 'chosen() === "carrier"'
              required:true
          li '.choice', ->
            label ->
              input
                type:'radio'
                name:name
                value:'gateway'
                bind:
                  checked: 'chosen'
                required:true
              text 'Use Gateway '
            input
              list:'gateway'
              name: 'gwid'
              bind:
                value: 'gwid'
                enable: 'chosen() === "gateway"'
              required: true

      ko.components.register 'rule-target',
        viewModel: view
        template: teacup.render html

      {RuleTarget,rule_target}

    teacup = require 'teacup'
    teacup.use (require 'teacup-databind')()
    assert = require 'assert'
