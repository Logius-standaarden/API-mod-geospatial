# Request and response

Providing requested resources is the essence of any API. This also applies to REST APIs that handle geospatial data. There are, however, some specific aspects when dealing with geospatial data in REST APIs. The most important aspects are described in this chapter:

- how to encode geometries in APIs
- how to supply a spatial filter in the call (request)
- how to return results of a spatial search

When requesting information, for example about cadastral parcels, users do not necessarily require the geometry, even if they used a spatial filter. A name or parcel ID may be sufficient.

<aside class="note">
The Geospatial Module is focused on JSON-based encoding of data. However, consider also supporting <code>text/html</code>, as recommended in OGC API Features [[ogcapi-features-1]]. Sharing data on the Web should include publication in HTML, as this allows discovery of the data through common search engines as well as viewing the data directly in a browser.
</aside>

## GeoJSON

[[rfc7946]] describes the GeoJSON format, including a convention for describing 2D geometric objects in CRS84 (OGC:CRS84). In the Geospatial module of the API strategy we adopt the GeoJSON conventions for describing geometry objects. The convention is extended to allow alternative projections.
The GeoJSON conventions and extensions described in this module apply to both geometry passed in input parameters and responses.

<aside class="note">
GeoJSON does not cover all use cases. For example, it is not possible to store circular arc geometries or solids in GeoJSON. In such cases, there are several valid options:

- Use alternative standardized formats for geospatial data, such as [WKT](https://www.w3.org/TR/sdw-bp/#dfn-well-known-text-(wkt)) or its binary equivalent WKB; GML [iso-19136-2007]; or in future [OGC JSON-FG](https://docs.ogc.org/DRAFTS/21-045.html) (currently a draft standard).
- When supporting GML, do this according to OGC API Features [Requirements class 8.4](https://docs.ogc.org/is/17-069r3/17-069r3.html#_requirements_class_geography_markup_language_gml_simple_features_profile_level_0) for GML Simple Features level 0, or [Requirements class 8.4](https://docs.ogc.org/is/17-069r3/17-069r3.html#_requirements_class_geography_markup_language_gml_simple_features_profile_level_2) for GML Simple Features level 2.
- Use a workaround, e.g. convert circular lines / arcs to regular linestrings.

</aside>

<p>Example of embedding WKT in a JSON object using the following definition for a JSON object:</p>
  <pre class="example">
  building:
    type: object
    required:
      - geometry
    properties:
      geometry:
        type: string
        format: wkt
  </pre>
<p>Sample response:</p>
  <pre class="example">
  {
    "building": {
      "geometry": "POLYGON Z((194174.445 465873.676 0, 194174.452 465872.291 0, 194158.154 465872.213 0, 194158.226 465856.695 0, 194223.89 465856.969 0, 194223.821 465872.48 0, 194207.529 465872.415 0, 194207.505 465882.528 0, 194207.498 465883.902 0, 194223.799 465883.967 0, 194223.732 465899.48 0, 194216.55 465899.45 0, 194215.15 465899.445 0, 194213.85 465899.439 0, 194158.068 465899.211 0, 194158.148 465883.685 0, 194174.42 465883.767 0, 194174.445 465873.676 0))"
    }
  }
  </pre>

<p>Example of embedding WKB in a JSON object using the following definition for a JSON object:</p>
  <pre class="example">
  building:
    type: object
    required:
      - geometry
    properties:
      geometry:
        type: string
        format: wkb
  </pre>
<p>Sample response:</p>
  <pre class="example">
  {
    "building": {
      "geometry": "01030000A0F71C00000100000012000000F6285C8FF3B30741105839B4466F1C4100000000000000000E2DB29DF3B307416DE7FB29416F1C4100000000000000001D5A643B71B3074108AC1CDA406F1C4100000000000000008716D9CE71B307417B14AEC7026F1C410000000000000000EC51B81E7FB50741378941E0036F1C410000000000000000B07268917EB50741B81E85EB416F1C4100000000000000001D5A643BFCB407418FC2F5A8416F1C410000000000000000A4703D0AFCB407413108AC1C6A6F1C4100000000000000008B6CE7FBFBB4074154E3A59B6F6F1C410000000000000000AC1C5A647EB507417D3F35DE6F6F1C410000000000000000E5D022DB7DB50741B81E85EBAD6F1C4100000000000000006666666644B50741CDCCCCCCAD6F1C4100000000000000003333333339B507417B14AEC7AD6F1C410000000000000000CDCCCCCC2EB507414C3789C1AD6F1C4100000000000000008195438B70B307414E6210D8AC6F1C410000000000000000BE9F1A2F71B30741D7A370BD6E6F1C410000000000000000C3F5285CF3B30741B07268116F6F1C410000000000000000F6285C8FF3B30741105839B4466F1C410000000000000000"
    }
  }
  </pre>

## Call (requests)

A simple spatial filter can be supplied as a bounding box. This is a common way of filtering spatial data and can be supplied as a parameter. We adopt the OGC API Features [[ogcapi-features-1]] bounding box parameter:

<div class="rule" id="/geo/bbox-query-parameter">
  <p class="rulelab"><b>/geo/bbox-query-parameter</b>: Supply a simple spatial filter as a bounding box parameter</p>
  <dl>
    <dt>Statement</dt>
    <dd>
      Support the <a href="https://docs.ogc.org/is/17-069r4/17-069r4.html#_parameter_bbox">OGC API Features part 1 <code>bbox</code> query parameter</a> in conformance to the standard.</p>
      <aside class="example">
        <code>GET /api/v1/buildings?bbox=5.4,52.1,5.5,53.2</code>
      </aside>
      <div class="note">Note that if a resource contains multiple geometries, it is up to the provider to decide if geometries of type single geometry or type multiple geometry are returned and that the provider shall clearly document this behavior.</div>
    </dd>
    <dt>Rationale</dt>
    <dd>
      <p>The default spatial operator <code>intersects</code> is used to determine which resources are returned.
      <p>Due to possible performance issue, especially when a combination of filters is used, a provider may decide to limit the size of the bounding box or the number of results. It is also up to the provider to decide if an error is returned in such cases. The provider shall clearly document this behavior.
      <p>The provider shall be able to provide resources that do not have a geometry property and are related to resources that match the bounding box filter.
      <p>An error shall be given if the provided coordinates are outside the specified coordinate reference system.
    </dd>
    <dt>How to test</dt>
    <dd>
      <ol>
        <li>Issue an HTTP GET request to the API, including the <code>bbox</code> query parameter and using <a href="#crs-negotiation">CRS Negotiation</a>.</li>
        <li>Validate that a response with status code 200 is returned.</li>
        <li>Verify that only features that have a spatial geometry that intersects the bounding box are returned as part of the result set.</li>
      </ol>
    </dd>
  </dl>
</div>

<aside class="note">
Spatial operations like <code>intersects</code> and <code>within</code> in combination with a filter geometry (e.g. <code>bbox</code>) or resource geometry containing long lines, may result in erroneous responses, since a straight line between two coordinates in a CRS is, depending on the CRS, not a straight line in reality. See the <a href="https://docs.geostandaarden.nl/crs/crs/#vormvastheid">explanation in the Handreiking CRS</a> (Dutch).
</aside>

<aside class="note">
Spatial filtering is an extensive topic. There are use cases for geospatial operators like <code>intersects</code> or <code>within</code>. Geospatial filters can be large and complex, which sometimes causes problems since <code>GET</code> may not have a payload (although supported by some clients).

More complex spatial filtering is not addressed in this module. A new API Design Rules module on filtering will address spatial as well as non-spatial filtering. [[ogcapi-features-3]] will provide input for this.

However, until the filtering module is written, the geospatial module retains rule <a href="#/geo/geometric-context">/geo/geometric-context</a> about dealing with results of a global spatial query. This rule may be moved to the filtering module at a later stage.
</aside>

<span name="api-38"></span>
<div class="rule" id="/geo/geometric-context">
  <p class="rulelab"><b>/geo/geometric-context</b>: Place results of a global spatial query in the relevant geometric context</p>
  <dl>
    <dt>Statement</dt>
    <dd>
      In case of a global query <code>/api/v1/_search</code>, results should be placed in the relevant geometric context, because results from different <a href="https://gitdocumentatie.logius.nl/publicatie/api/adr/2.0.2/#resources">collections</a>, i.e. different sets of resources of the same type, are retrieved. Express the name of the collection to which the results belong in the singular form using the property <code>type</code>. For example:
      <aside class="example">
        <pre><code class="json">
// POST /api/v1/_search:
{
  "currentPage": 1,
  "nextPage": 2,
  "pageSize": 10,
  "_embedded": {
    "items": [
      {
        "type": "enkelbestemming",
        "_links": {
          "self": {
            "href": "https://api.example.org/v1/enkelbestemmingen/1234"
          }
        }
      },
      {
        "type": "dubbelbestemming",
        "_links": {
          "self": {
            "href": "https://api.example.org/v1/dubbelbestemmingen/8765"
          }
        }
      }
    ]
  }
}
        </code></pre>
      </aside>
    </dd>
    <dt>How to test</dt>
    <dd>
      <ol>
        <li>Issue an HTTP GET request to the API.</li>
        <li>Validate that the returned document contains the expected <code>type</code> property for each member.</li>
      </ol>
    </dd>
  </dl>
</div>

In case a REST API shall comply to the OGC API Features specification for creating, updating and deleting a resource, the following applies.

<span name="api-34"></span>
<div class="rule" id="/geo/geojson-request">
  <p class="rulelab"><b>/geo/geojson-request</b>: Support GeoJSON in geospatial API requests</p>
  <dl>
    <dt>Statement</dt>
    <dd>
      For representing geometric information in an API, use the convention for describing geometry as defined in the GeoJSON format [[rfc7946]]. Support GeoJSON as described in <a href="https://docs.ogc.org/DRAFTS/20-002r1.html">OGC API Features part 4</a>, but note that this standard is still in development.
      <aside class="example">
        POST feature
        <pre><code class="json">
// POST /collections/gebouwen/items   HTTP/1.1
// Content-Type: application/geo+json
{
  "type": "Feature",
  "geometry":  {
    "type": "Point",
    "coordinates": [5.2795,52.1933]
  },
  "properties": {
    "naam": "Paleis Soestdijk",
    // ...
  }
}
        </code></pre>
      </aside>
      <aside class="example">
        POST feature collection
        <pre><code class="json">
// POST /collections   HTTP/1.1
// Content-Type: application/geo+json
{
  "type": "FeatureCollection",
  "features": [
  {
    "type": "Feature",
    "geometry":  {
      "type": "Point",
      "coordinates": [5.2795,52.1933]
    },
    "properties": {
      "naam": "Paleis Soestdijk",
      // ...
    }
  }]
}
        </code></pre>
      </aside>
    </dd>
    <dt>How to test</dt>
    <dd>
      <ol>
        <li>Create a new resource that includes feature content (i.e. coordinates) using the HTTP POST method with request media type <code>application/geo+json</code> in the <code>Content-Type</code> header.</li>
        <li>Validate that a response with status code <code>201</code> (Created) is returned.</li>
        <li>Validate that the response includes the <code>Location</code> header with the URI of the newly added resource.
      </ol>
    </dd>
  </dl>
</div>

In case a REST API does not have to comply to the OGC API Features specification, e.g. for usage in administrative applications, the REST API shall use the JSON data format. If a resource contains geometry, that geometry shall be embedded as a GeoJSON <code>Geometry</code> object within the resource. The media type <code>application/json</code> must be supported. This may also apply to other media types <code>application/*+json</code>, however this depends on the media type specification. If the media type specification prescribes that resource information must be embedded in a JSON structure defined in the media specification, then the media type should not be supported while it is impossible to comply to that specification with the method described below. The media type <code>application/geo+json</code> should not be supported while the resource does not comply to the GeoJSON specification, i.e. the request resource does not embed a feature or feature collection.
A template for the definition of the schemas for the GeoJSON <code>Geometry</code> object in the requests in OpenAPI definitions is available: [geometryGeoJSON.yaml](https://schemas.opengis.net/ogcapi/features/part1/1.0/openapi/schemas/geometryGeoJSON.yaml).
In case a collection of resources is embedded in the request resource, the name of the array containing the resources should be the plural of the resource name.

<div class="rule" id="/geo/embed-geojson-geometry-request">
  <p class="rulelab"><b>/geo/embed-geojson-geometry-request</b>: Embed GeoJSON <code>Geometry</code> object as part of the JSON resource in API requests</p>
  <dl>
    <dt>Statement</dt>
    <dd>
      When a JSON (<code>application/json</code>) request contains a geometry, represent it in the same way as the <code>Geometry</code> object of GeoJSON.
      <aside class="example">
        POST resource containing geometry
        <pre><code class="json">
// POST /collections/gebouwen/items   HTTP/1.1
// Content-Type: application/json
{
  "naam": "Paleis Soestdijk",
  "geometrie": {
    "type": "Point",
    "coordinates": [5.2795,52.1933]
  }
}
        </code></pre>
      </aside>
      <aside class="example">
        POST resource containing geometry collection
        <pre><code class="json">
// POST /collections/gebouwen/items   HTTP/1.1
// Content-Type: application/json
{
  "naam": "Paleis Soestdijk",
  "geometrie": {
    "type": "GeometryCollection",
    "geometries": [
      {
        "type": "Point",
        "coordinates": [5.2795,52.1933]
      }
    ]
  }
}
        </code></pre>
      </aside>
    </dd>
    <dt>How to test</dt>
    <dd>
      <ol>
        <li>Create a new resource that includes geometry of GeoJSON <code>Geometry</code> object type using the HTTP POST method with request media type <code>application/json</code> in the <code>Content-Type</code> header.</li>
        <li>Validate that a response with status code <code>201</code> (Created) is returned.</li>
        <li>Validate that the response includes the <code>Location</code> header with the URI of the newly added resource.
      </ol>
    </dd>
  </dl>
</div>

## Result (response)

In case a REST API shall comply to the OGC API Features specification, e.g. for usage in GIS applications, the following applies.

<div class="rule" id="/geo/geojson-response">
  <p class="rulelab"><b>/geo/geojson-response</b>: Support GeoJSON in geospatial API responsess</p>
  <dl>
    <dt>Statement</dt>
    <dd>
      For representing 2D geometric information in an API response, use the convention for describing geometry as defined in the GeoJSON format [[rfc7946]]. Support GeoJSON as described in OGC API Features <a href="https://docs.ogc.org/is/17-069r3/17-069r3.html#_requirements_class_geojson">Requirements class 8.3</a> [[ogcapi-features-1]].
      <aside class="example">
        Feature
        <pre><code class="json">
// GET /collections/gebouwen/items/0308100000022041   HTTP 1.1
// Content-type: application/geo+json
{
  "type": "Feature",
  "id": "0308100000022041",
  "geometry":  {
    "type": "Point",
    "coordinates": [5.2795,52.1933]
  },
  "properties": {
    "naam": "Paleis Soestdijk",
    // ...
  },
  "links": [
    {
      "self": "/collections/gebouwen/items/0308100000022041"
    }
  ]
}
        </code></pre>
      </aside>
      <aside class="example">
        Feature collection
        <pre><code class="json">
// GET /collections/gebouwen   HTTP 1.1
// Content-type: application/geo+json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "0308100000022041",
      "geometry":  {
        "type": "Point",
        "coordinates": [5.2795,52.1933]
      },
      "properties": {
        "naam": "Paleis Soestdijk",
        // ...
      },
      "links": [
        {
          "self": "/collections/gebouwen/0308100000022041"
        }
      ]
    },
    {
    }
  ],
  "timeStamp" : "2023-02-22T10:32:23Z",
  "numberMatched" : "0308100000022041",
  "numberReturned" : "1",
  "links": [
    {
      "self": "/collections/gebouwen"
    },
    {
      "next": ""
    }
  ]
}
        </code></pre>
      </aside>
      <div class="note">
        <li>The resources' properties (e.g. <code>naam</code>) are passed in the properties object. Depending on the implemented filter capabilities the properties object may contain all or a selection of the resources' properties.</li>
        <li>The OGC API Features specification provides the possibility to add an array of links to a feature and feature collection, which may contain a self link and in case of a feature collection may contain navigation links</li>
      </div>
    </dd>
    <dt>How to test</dt>
    <dd>
      Test case 1:
      <ol>
        <li>Request a single resource that includes feature content (i.e. coordinates) with response media type <code>application/geo+json</code> in the <code>Accept</code> header.</li>
        <li>Validate that a response with status code 200 is returned.</li>
        <li>Validate that <code>Content-Type</code> header contains <code>application/geo+json</code></li>
        <li>Validate that the returned document is a GeoJSON Feature document.</li>
      </ol>
      Test case 2:
      <ol>
        <li>Request a collection of resources that includes feature content (i.e. coordinates) with response media type <code>application/geo+json</code> in the <code>Accept</code> header.</li>
        <li>Validate that a response with status code 200 is returned.</li>
        <li>Validate that <code>Content-Type</code> header contains <code>application/geo+json</code></li>
        <li>Validate that the returned document is a GeoJSON FeatureCollection document.</li>
      </ol>
      Test case 3:
      <ol>
        <li>Request a single resource that does not include feature content (i.e. coordinates) with response media type <code>application/geo+json</code> or <code>application/json</code> in the <code>Accept</code> header.</li>
        <li>Validate that a response with status code 200 is returned.</li>
        <li>Validate that <code>Content-Type</code> header contains <code>application/json</code></li>
        <li>Validate that the returned document is a JSON document.</li>
      </ol>
      Test case 4:
      <ol>
        <li>Request a collection of resources that do not include feature content (i.e. coordinates) with response media type <code>application/geo+json</code> or <code>application/json</code> in the <code>Accept</code> header.</li>
        <li>Validate that a response with status code 200 is returned.</li>
        <li>Validate that <code>Content-Type</code> header contains <code>application/json</code></li>
        <li>Validate that the returned document is a JSON document.</li>
      </ol>
    </dd>
  </dl>
</div>

In case a REST API does not have to comply to the OGC API Features specification, e.g. for usage in administrative applications, the REST API shall use the JSON data format. If resources contain geometry, the geometry shall be returned as a GeoJSON <code>Geometry</code> object embedded in the resource. The media type <code>application/json</code>  must be supported. This may also apply to other media types <code>application/\*+json</code>, however this depends on the media type specification. If the media type specification prescribes that resource information must be embedded in a JSON structure defined in the media type specification, then the media type should not be supported while it is impossible to comply to that specification with the method described below. The media type <code>application/geo+json</code> should not be supported while the resource does not comply to the GeoJSON specification, i.e. the response does not return a feature or feature collection.
A template for the definition of the schemas for the GeoJSON <code>Geometry</code> object in the responses in OpenAPI definitions is available: [geometryGeoJSON.yaml](https://schemas.opengis.net/ogcapi/features/part1/1.0/openapi/schemas/geometryGeoJSON.yaml).
In case a collection of resources is returned, the name of the array containing the resources should be the plural of the resource name.

<span name="api-35"></span>
<div class="rule" id="/geo/embed-geojson-geometry-response">
  <p class="rulelab"><b>/geo/embed-geojson-geometry-response</b>: Embed GeoJSON <code>Geometry</code> object as part of the JSON resource in API responses</p>
  <dl>
    <dt>Statement</dt>
    <dd>
      When a JSON (<code>application/json</code>) response contains a geometry, represent it in the same way as the <code>Geometry</code> object of GeoJSON.
      <aside class="example">
        Resource containing geometry
        <pre><code class="json">
// GET /gebouwen/0308100000022041   HTTP 1.1
// Content-type: application/hal+json
{
  "identificatie": "0308100000022041",
  "naam": "Paleis Soestdijk",
  "geometrie":  {
    "type": "Point",
    "coordinates": [5.2795,52.1933]
  },
  "_links": {
    {
      "self": "/gebouwen/0308100000022041"
    }
  }
}
        </code></pre>
      </aside>
      <aside class="example">
        Resource containing geometry collection
        <pre><code class="json">
// GET /gebouwen/0308100000022041   HTTP 1.1
// Content-type: application/hal+json
{
  "identificatie": "0308100000022041",
  "naam": "Paleis Soestdijk",
  "geometrie": {
    "type": "GeometryCollection",
    "geometries": [
      {
        "type": "Point"
        "coordinates": [5.2795,52.1933]
      },
      {
        "type": "Polygon"
        "coordinates" : [/* ... */]
      }
    ]
  },
  "_links": {
    {
      "self": "/gebouwen/0308100000022041"
    }
  }
}
        </code></pre>
      </aside>
      <aside class="example">
        Collection of resources containing geometry
        <pre><code class="json">
// GET /gebouwen   HTTP 1.1
// Content-type: application/hal+json
{
  "gebouwen": [
    {
      "identificatie": "0308100000022041",
      "naam": "Paleis Soestdijk",
      "geometrie":  {
        "type": "Point",
        "coordinates": [5.2795,52.1933]
      }
      "_links": {
        {
          "self": "/gebouwen/0308100000022041"
        }
      }
    }
  ],
  "_links": {
    {
      "self": "/gebouwen"
    },
    {
      "next": ""
    }
  }
}
        </code></pre>
      </aside>
      <p class="note">The resource and resource collection may be [[HAL]] resources and therefore may contain a `_links` object. The `_links` object should contain a self link and in case of a collection also navigation links (e.g. first, next prev, last). In such cases the <code>application/hal+json</code> media type may be used.
    </dd>
    <dt>How to test</dt>
    <dd>
      Test case 1:
      <ol>
        <li>Request a single resource that contains geometry of GeoJSON <code>Geometry</code> object type: <code>Point</code>, <code>MultiPoint</code>, <code>LineString</code>, <code>MultiLineString</code>, <code>Polygon</code> or <code>MultiPolygon</code> and with response media type <code>application/json</code> in the <code>Accept</code> header.</li>
        <li>Validate that a response with status code 200 is returned.</li>
        <li>Validate that <code>Content-Type</code> header contains <code>application/json</code></li>
        <li>Validate that the returned document is a JSON document.</li>
        <li>Validate that the returned document contains a property that complies to one of the GeoJSON <code>Geometry</code> objects mentioned above and contains:
        <ul>
          <li>a property <code>type</code> containing the name of one of the GeoJSON <code>Geometry</code> object types mentioned above, and</li>
          <li>a property <code>coordinates</code> containing an array with the coordinates. Depending on the type of geometry object, the content of the array differs.</li>
        </ul></li>
      </ol>
      Test case 2:
      <ol>
        <li>Request a collection of resources that contain geometry of GeoJSON <code>Geometry</code> object type: <code>Point</code>, <code>MultiPoint</code>, <code>LineString</code>, <code>MultiLineString</code>, <code>Polygon</code> or <code>MultiPolygon</code> and with response media type <code>application/json</code> in the <code>Accept</code> header.</li>
        <li>Validate that a response with status code 200 is returned.</li>
        <li>Validate that <code>Content-Type</code> header contains <code>application/json</code></li>
        <li>Validate that the returned document is a JSON document.</li>
        <li>Validate that the returned document contains an array of resources and that each resource contains a property that complies to one of the GeoJSON <code>Geometry</code> objects mentioned above and contains:
        <ul>
          <li>a property <code>type</code> containing the name of one of the GeoJSON <code>Geometry</code> object types mentioned above, and</li>
          <li>a property <code>coordinates</code> containing an array with the coordinates. Depending on the type of geometry object, the content of the array differs.</li>
        </ul></li>
      </ol>
      Test case 3:
      <ol>
        <li>Request a single resource that contains geometry of GeoJSON <code>Geometry</code> object type: <code>GeometryCollection</code> and with response media type <code>application/json</code> in the <code>Accept</code> header.</li>
        <li>Validate that a response with status code 200 is returned.</li>
        <li>Validate that <code>Content-Type</code> header contains <code>application/json</code></li>
        <li>Validate that the returned document is a JSON document.</li>
        <li>Validate that the returned document contains a property that complies to the GeoJSON <code>Geometry</code> object mentioned above and contains:
        <ul>
          <li>a property <code>type</code> containing the name of the GeoJSON <code>Geometry</code> object type: <code>GeometryCollection</code>, and</li>
          <li>a property <code>geometries</code> containing an array of GeoJSON <code>Geometry</code> objects.</li>
        </ul></li>
      </ol>
      Test case 4:
      <ol>
        <li>Request a collection of resources that contain geometry of GeoJSON <code>Geometry</code> object type: <code>GeometryCollection</code> and with response media type <code>application/json</code> in the <code>Accept</code> header.</li>
        <li>Validate that a response with status code 200 is returned.</li>
        <li>Validate that <code>Content-Type</code> header contains <code>application/json</code></li>
        <li>Validate that the returned document is a JSON document.</li>
        <li>Validate that the returned document contains an array of resources and that each resource contains a  property that complies to the GeoJSON <code>Geometry</code> object mentioned above and contains:
          <ul>
            <li>a property <code>type</code> containing the name of the GeoJSON <code>Geometry</code> object type: <code>GeometryCollection</code>, and</li>
            <li>a property <code>geometries</code> containing an array of GeoJSON <code>Geometry</code> objects.</li>
          </ul>
        </li>
      </ol>
    </dd>
  </dl>
</div>
