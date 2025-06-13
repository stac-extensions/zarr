# Zarr Extension Specification

- **Title:** Zarr Extension Specification
- **Identifier:** <https://jsignell.github.io/zarr/v1.0.0/schema.json>
- **Field Name Prefix:** zarr
- **Scope:** Asset
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @jsignell

This document explains the Zarr Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

This extension helps users open STAC Assets pointing to Zarr stores. It includes fields from Zarr metadata that are
relevant when opening a Zarr store. The goal of this extension is not to reproduce all Zarr metadata within STAC.

This extension takes inspiration from the deprecated [Xarray Assets Extension](https://github.com/stac-extensions/xarray-assets)
by @TomAugspurger and can be used as a replacement when paired with the Storage Extension and `xpystac` (see Python Example below).

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [ ] Collections
- [ ] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name           | Type                      | Description                                  |
| -------------------- | ------------------------- | -------------------------------------------- |
| zarr:consolidated    | boolean                   | Whether the Zarr store includes consolidated metadata |
| zarr:node_type       | string                    | Type of Zarr hierarchy node element. Must be `group` or `array` |
| zarr:zarr_format     | integer                   | Zarr format of the store (currently 2 or 3)  |

### Additional Field Information

#### zarr:consolidated

[Consolidated metadata](https://zarr.readthedocs.io/en/main/user-guide/consolidated_metadata.html)
stores all the metadata for a Zarr hierarchy in the metadata of the root Group. This boolean 
`consolidated` fields is useful when opening a Zarr store in xarray: 
`xarray.open_dataset(... consolidated=True)`.

A value of `true` indicates that:

For Zarr 2: there is a top-level .zmetadata document.
For Zarr 3: within the top-level Zarr metadata there is a `consolidated_metadata` field.

Note: Consolidated metadata is not officially part of the Zarr specification
([PR to add it](https://github.com/zarr-developers/zarr-specs/pull/309)),
but it is useful to know whether or not it is present when opening a Zarr store.

#### zarr:node_type

As defined in the [Zarr v3 Specification](https://zarr-specs.readthedocs.io/en/latest/v3/core/index.html)

> A string defining the type of hierarchy node element

`node_type` indicates which data model to use when opening the Zarr store. For `group` a tree structure is recommended 
(such as `xarray.DataTree`). For array an array structure (such as `xarray.DataArray`). Note that xarray does not
currently support reading Zarr Arrays directly into `xarray.DataArray` objects, but other libraries 
such as GDAL, zarrs, and zarr-python do.

#### zarr:zarr_format

As defined in the [Zarr v3 Specification](https://zarr-specs.readthedocs.io/en/latest/v3/core/index.html):

> An integer defining the version of the storage specification to which the array store adheres.

`zarr_format` has implications with respect to the versions of libraries required to open the Zarr store.

## Python Example

This extension will be used by [xpystac](https://github.com/stac-utils/xpystac) to enable the following:

```python
import planetary_computer
import pystac_client
import xarray as xr


catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)

collection = catalog.get_collection("daymet-daily-hi")
asset = collection.assets["zarr-abfs"]

xr.open_dataset(asset, patch_url=planetary_computer.sign)
```

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
