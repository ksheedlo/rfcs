* Start Date: 2018-07-23
* RFC PR: (leave this empty)
* Fusion Issue: (leave this empty)

## Summary

Two new plugins, `fusion-plugin-geoip` and `fusion-plugin-geoip-react`. They
can be used to geolocate the user based on their IP address and provide IP
geolocation data to React components respectively.

## Basic example

### fusion-plugin-geoip

```js
const geoipData = GeoIP.from(ctx);
console.log(geoipData.countryCode);
console.log(geoipData.lat);
console.log(geoipData.lng);
```

### fusion-plugin-geoip-react

```js
import {withGeoip} from 'fusion-plugin-geoip-react';

function HelloGeoip({geoip}) {
  return <p>Hello {geoip.countryName}</p>;
}

const HelloGeoipComponent = withGeoip(HelloGeoip);
```

## Motivation

These plugins will support GeoIP, which is a common requirement for websites.
For instance, GeoIP could be used to determine whether to show an EU cookie
banner. Other potential use cases include CMS sites showing different content
based on the user's country or map applications wanting to show a relevant
section of the world to each user.

The end goal is to have plugins that site builders can install, configure with
a Maxmind database file, and immediately start using geoip in their Fusion
application.
	
## Detailed design

### fusion-plugin-geoip

We will use `@lxe/maxmind-db-reader` to load the geoip data from `ctx.ip` on
the server. A `geoip data` is a specific object that we define here using a
Flow type. The Maxmind DB reader needs to be initialized with a Maxmind .mmdb
file, so we additionally define a `GeoipConfigToken`. The types are as follows:

```js
type GeoipConfig = {
  dbPath: string,
  defaultGeoData?: GeoipData
};

type GeoipData = {
  continentCode: string,
  continentName: string,
  countryCode: string,
  countryName: string,
  cityName: ?string,
  lat: number,
  lng: number,
  timeZone: string, // e.g., "America/Los_Angeles"
};
```

The type of `GeoIP.from` is `(ctx: Context) => GeoipData` and it will be
memoized. The `dbPath` is required and the server should not start if it is not
a valid path to a .mmdb database file. All consumers use `GeoIP.from` to load
geoip data from the context. If geoip data is not available, then the user has
the option to configure `defaultGeoData` that will be used instead. If geoip
data is not available and the user does not configure `defaultGeoData`, geoip
will resolve to a default value chosen by the package. This is acceptable
because GeoIP is generally understood to be a best-effort estimation and can
never achieve 100% accuracy.

We will additionally define a middleware that includes the geoip data in JSON
in a script tag on the page. We will then load this tag in the browser in
order to provide geoip there. This design is similar to how
fusion-plugin-i18n works.

### fusion-plugin-geoip-react

We will set up a context provider that provides GeoIP data, and a `withGeoip`
HOC that consumes the geoip data from React context. The HOC will inject the
`geoip` prop containing geoip data.

## Drawbacks

The fusion team would have to support two more plugins. The feature could be
implemented in user space, albeit with significant duplication. The impact on
teaching people Fusion should be small, since geoip is a relatively advanced
feature and probably will not be included in create-fusion-app. There is no
cost to migrate existing Fusion applications to it, unless they have already
implemented their own geoip, in which case they would just need to swap out a
plugin. It does not have any integration points with existing open-source
fusion packages, which helps reduce complexity.

The trickiest part of owning this package will probably be owning the setup and
API documentation. We will need to clearly document that users need to bring
their own Maxmind database file.

## Alternatives

If we don't do this, then folks building websites in Fusion will end up
duplicating geoip functionality. Maybe someone out in the community will
eventually end up building geoip instead.

## Adoption strategy

We will add API documentation and perhaps also a user guide to the fusion
website. GeoIP will not be added to create-fusion-app, since it is not needed
on many websites and requires the Maxmind database file. Adding the packages is
not a breaking change.

## How we teach this

I think "geoip data" is the most appropriate word for the result of looking up
a request. All other concepts described in this RFC should fit well with
existing fusion patterns.

If this proposal is accepted, we will need to add 3 pages to the fusion
website. `fusion-plugin-geoip` and `fusion-plugin-geoip-react` will each have
an API docs page, and then we should write a user guide that describes how to
use geoip altogether.

This feature only needs to be taught to Fusion developers who need to use it,
so adding documentation along with a short blog article or tweeting it out
should be enough.

## Unresolved questions

* Naming. `lat`, `lng` or `latitude`, `longitude`? Other suggestions on how to name things are welcome.

