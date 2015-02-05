A rule target Knockout Widget for CCNQ(4)
-----------------------------------------

A rule target is an item in the `gwlist` field of a `rule` entry in a CCNQ4 `ruleset` database. Such a database is used by the [`tough-rate`](https://github.com/shimaore/tough-rate) LCR engine, especially its `routes-registrant`, `routes-carrierid`, `routes-gwid` middleware modules.

This module adds a `rule-target` component in Knockout.

Usage
-----

```javascript
require('ccnq-ko-rule-target')(knockout);
```

```html
<rule-target bind-data="params: data: ..., gateways: ..., carriers: ..."></rule-target>
```

Parameters:
- data: the `gwlist` item
- gateways: a list of valid gateways
- carriers: a list of valid carriers

    module.exports = (ko) ->

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

          # Behaviors
          return

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

Flow the data back to the model.

        @chosen.subscribe =>
          value.source_registrant @chosen() is 'registrant'

        @gwid = value.gwid
        @carrierid = value.carrierid

        gateway_valid = (id) -> id? and id in gateways
        carrier_valid = (id) -> id? and id in carriers

        @valid = ko.pureComputed =>
          is_valid = switch @chosen()
            when 'registrant'
              true
            when 'gateway'
              gateway_valid @gwid()
            when 'carrier'
              carrier_valid @carrierid()
            else
              false

Flow the data back to the model.

          value._validated is_valid # Flow back to the data.
          is_valid

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

      RuleTarget

    teacup = require 'teacup'
    teacup.use (require 'teacup-databind')()
    assert = require 'assert'
