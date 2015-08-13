# Ideal Integratoin Service
<table>
  <tr>
    <td nowrap>tenant lease date </td>
    <td>no</td>
    <td>asdlfjalsdjf asdlfjas dlfjasd lfajsdf lkajsdflka jsdlfajksdlf jasdlfjasdl jflkads jflkajsdf lakjsdflka sjdflkadsjf</td>
  </tr>
</table>
Single REST like service that allows us to query a list of leases to get leases for tentants who will be moving in (future tense), moving out, or transfering.

## Query options
The service would include the ability to query the leases to be able to filter on the below

1. start and end date of the lease
2. status of the lease
   a. status(es) that indiciate that the lease is signed but the tenants have not yet moved in
   b. status(es) that indicate that the lease is ending and the tenants will be moving out of the apartment
   c. status(es) for transfers from 1 apartment/home to another if your applicable for your system
3. client id
4. property id

The actual params/filters do not matter. We just need a way to retrieve the data based on the above.

## HTTP Method
The service could be a GET or a POST. Although GET is more appropriate for REST it this case it does nto matter. 

If GET is chosen, since the status param may have multiple values it's best practice to follow the below convention

For a single status
`https://youservice.com/leases?status=approved`

For multiple statuses
`https://youservice.com/leases?status=[approved,ending]`

It's not ideal to use `status=approved&status=ending` since many REST clients & servers will only send or read the last value.


## Service Response
Here's an example of an ideal response of the individual lease record (this excludes a higher level object for paging, etc if paging is going to be implemented)


```javascript
{
  "id": "asdfl230as8", // The id of the lease
  "client_id": "1234",
  "property_id": "5678",
  "unit_address": {
    "street": "1140 Broadway",
    "apt": "905",
    "city": "New York",
    "state": "NY",
    "zip": "10010"
  },
  "status": "approved",
  "lease_start_date": "2015-11-01",
  "lease_end_date": "2016-10-31",

  // Field to indicate the units use such as corporate housing, vacation, etc
  "use_type": "coporate",

  // Only applicable on move-ins
  "move_in_date": "2015-11-01",

  // Only applicable on move-outs
  "move_out_date": "2015-10-11",


  "tenants": [
    {
      "id": "6789"  // The individual tenant or contact id
      "first_name: "Ryan",
      "last_name": "Hubbard",
      "email": "ryan@updater.com",
      "cell_phone": "555-212-4567",
      "home_phone": "555-212-3456",
      "gender": "male",

      // The tenants previous address
      // Only applicable for move-ins
      "previous_address": {
        "street": "28 W 22nd Street",
        "apt": "2G",
        "city": "New York",
        "state": "NY",
        "zip": "10010"
      },

      // Where the tenant is moving to
      // Only applicable for move-outs
      "forwarding_address": {
        "street": "30 W 48nd Street",
        "apt": "12K",
        "city": "New York",
        "state": "NY",
        "zip": "10043"
      }
    },
    {
      ...
    }
  ]
}
```

## Important Notes on the Response
Above is just an ideal response. We can handle virtually any response. For example:

1. The field names could be anything
2. We prefer JSON but you could use CSV, XML, etc
2. Tenants is preferred as a nested object (array) but can be flattened
3. Sometimes the previous & forwarding address is kept on the lease level instead of the tenant level

## Response Fields

Name | Required | Description
---- | -------- | -----------
id | yes | The id of the lease
client id | yes | The id of your client
property id | yes | the id of the property. this needs to be unique within each client
unit address | yes | the address of the unit
status | yes | the status of the lease which will indicate whether they are moving in, out or transferring. This doesn't have to be a single field. If multiple fields are used in order to indicate a move in, out or transfer that's completely ok
move date | no | If you know when the tenants will be moving in or out. This can be a single field such as `move_date` or you may use multiple fields such as `move_in_date` and `move_out_date`
lease start date | yes | the start of the lease. In the case of a move-in, if no move date is provided this will be used as the tenants move in date
lease end date | yes | the end date of the lease. In the case of a move-out, if no move date is provided whit will be used as the tenants move out date
tenant first name | yes |
tenant last name | yes |
tenant email | yes |
tenant cell phone | no | Although it's not required it's highly recommended for SMS based invites
tenant home phone | no | Although it's highly recommended
tenant gender | no | the tenants gender (male|femalse|m|f)
tenant previous address | no | The tenants address they are currently at before they move-in. This address is only used for move ins and is ignored for move-outs or transfers. Although it's highly recommended as it will prefill this info for the tenant when signing up
tenant forwarding address | no | The tenants address that they are moving to when they move-out. This address is only used for move outs (and maybe used for transfers as well). Although it's highly recommended as it will prefill this info for the tenant when signing up


### Filtering
We can create filters to filter any transactions that should ignored on our platform. The fitler can use any fields in the response. Therefore any extra information about the lease or tenant that you or your client may want to create a filter on should be included. 

For example you may not want to send invites to corporate housing, so you should include which ever field has this information about the lease.

### Dates

We can accept virtually any date format although we prefer `YYYY-MM-DD`

### Phones

Some systems don't designate between cell phone and land lines. You can just send phone fields as `phone` or `phone1`, `phone2`. Our technology will determine if it's a cell phone or land line.
