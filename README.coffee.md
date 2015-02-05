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
        constructor: ({data,$root}) ->
          {gateways,carriers} = $root
          assert data?, 'data is required'
          assert gateways?, 'gateways is required'
          assert carriers?, 'carriers is required'

Data
----

A `gwlist` item typically contains one of:
- `source_registrant` -- if the rule should route through the caller's registrant;
- `gwid` -- if the call is to be routed out through a given gateway;
- `carrierid` -- if the call is to be routed out through a carrier (a set of gateways with similar costs).

          chosen = if data.source_registrant is true
              'registrant'
            else if data.carrierid?
              'carrier'
            else if data.gwid?
              'gateway'
          @chosen = ko.observable chosen

          @source_registrant = ko.pureComputed => @chosen() is 'registrant'
          @gwid = ko.observable data.gwid
          @carrierid = ko.observable data.carrierid

          gateway_valid = -> (not @gwid()?) or @gwid() in gateways
          carrier_valid = -> (not @carrierid()?) or @carrierid() in carriers
          @valid = ko.pureComputed =>
            switch @chosen()
              when 'registrant'
                true
              when 'gateway'
                gateway_valid()
              when 'carrier'
                carrier_valid()

FIXME: We need a way to let the component's user know whether the data is valid or not!

          # Behaviors
          return

      html = ->
        name = "rule-target-#{Math.random()}"
        {a,ul,li,label,input,text} = teacup
        ul '.target', bind: visible: 'chosen() !== "none"', ->
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
        viewModel: RuleTarget
        template: teacup.render html

      RuleTarget

    teacup = require 'teacup'
    teacup.use (require 'teacup-databind')()
    assert = require 'assert'
