---
layout: post
title:  "Discovering and Accessing Data at CEDA: The way forward with DataPoint"
author: Daniel Westwood
date:   2025-06-24 15:00:00
tags: [Cloud, API, STAC, Kerchunk, Zarr, Interoperable, Pystac]
---

TL;DR The CEDA DataPoint API has now been added to the JASMIN Standard Computing Environment (JASPY) Module. Users across all fields can discover data represented in STAC using this new API that is specific to their areas of study/research, and access datasets with ease regardless of underlying data format.

For several years, CEDA has been working towards a STAC-based metadata catalog which represents a large set of our data holdings within the archive. STAC (Spatio-Temporal Asset Catalog) is a widely adopted standard for metadata records within the Earth Observation community and beyond, created as a standardised way to expose collections of spatio-temporal data. The CEDA STAC catalog has been in development for several years and has become part of the Earth Observation Data Hub (EODH) project led by the National Centre for Earth Observation (NCEO). 

DataPoint is a package developed at CEDA to provide key functionality for improved search functionality across the CEDA STAC catalog, as well as integrating known cloud-optimised formats for effective access to analysis-ready data. In this article we discuss the inspiration for creating DataPoint and its impact for the user community, as well as the strategic direction for CEDA towards cloud-enabled data processing. 

# Index
- [Acronyms](#acronyms)
- [Cloud Optimised Formats for Data](#cloudformats)
- [Inspiration for DataPoint](#inspiration)
- [DataPoint API Features](#features)
- [Current CEDA STAC Collections](#collections)
- [Next Steps](#next-steps)
- [Getting in touch](#getting-in-touch)

# Acronyms

| Acronym | Meaning |
| ------- | ------- |
| STAC | Spatio Temporal Asset Catalog |
| CMIP | Coupled Model Intercomparison Project |
| WCRP | World Climate Research Project |
| ARD | Analysis Ready Data |
| EOCIS | Earth Observation Climate Information Service |
| EODH | Earth Observation Data Hub |

# Cloud Optimised Formats for Data

Over the last few years CEDA has developed various tools to generate new cloud-optimised data representations, with tools like Kerchunk and Zarr API. These tools have been developed to enable greater access to large sets of data traditionally stored in data archives that require knowledge of the filesystem and access methods in order to make use of the data. Cloud-optimised data representations open the door to greater interconnectivity between data centres and fewer physical/technical barriers to research requiring data analysis. CEDA focuses specifically on data representations/references using Kerchunk, where duplication of data for cloud-based storage is minimised, as cloud storage can often be expensive and constitute a considerable additional carbon cost.

{% include figure.html
    image_url="assets/img/posts/2025-06-24-ceda-datapoint/KerchunkAccess.png"
    description="Figure 1: Kerchunk Access"
%}

{% include figure.html
    image_url="assets/img/posts/2025-06-24-ceda-datapoint/ZarrAccess.png"
    description="Figure 2: Zarr Cloud-based Data Access"
%}

These new data representations have been ingested into the CEDA Archive and are accessible to all users - except that no one knows how to use or even find them. Since these technologies are relatively new to most of the communities that make use of our services, the mechanisms for accessing these types of data are not well known or well understood. Researchers using our data are more familiar with well-established formats like NetCDF/HDF which have existed for several decades as standards for data archival. What has been missing is a collation of these technologies into a single tool that CEDA can direct users towards, and focus efforts towards improving data access across all new developments.

# Inspiration for DataPoint

## Why did we create DataPoint?

DataPoint is the culmination of several projects involving the creation of a single point of access to the data archived at CEDA; the so-called ‘CEDA Singularity’. It is a STAC-based API client which connects to our STAC catalogs and can be used to search across our data holdings to find specific datasets and metadata. What sets DataPoint apart is the ability to directly open datasets from cloud formats without the user needing to understand the format or specific configuration for opening the dataset.

{% include figure.html
    image_url="assets/img/posts/2025-06-24-ceda-datapoint/SearchFutures.png"
    description="Figure 3: DataPoint pulls together STAC records and Cloud-optimised file formats"
%}

## Why should you use DataPoint?

Content

# DataPoint API Features

DataPoint has been designed with scientific research use cases in mind - with encouragement for feedback from the user community at every stage that can be incorporated into new feature releases for DataPoint. Highlighted here are some of the features added to DataPoint that are specifically aimed at improving the user experience of finding and accessing data.

## Abstraction of the Filesystem

STAC assets within the collections that are accessed by DataPoint have a reference to the exact data path within the CEDA archive where specific files can be found. This is used within DataPoint to locate the specific asset file, without the user needing to know the location of relevant datasets themselves. The creation of data cubes (objects that contain metadata about the datasets) is seamlessly abstract, such that no user-supplied information should be necessary, beyond the search parameters needed to identify the correct dataset. This represents a shift from a **file-based** to **object-based** mindset, although DataPoint also makes it easy to extract the set of assets as a list for any given dataset - if users prefer to locate the files themselves.

## Format-Agnostic Data Handling

Where DataPoint objects have been created to represent specific datasets, these objects have the capacity to access and serve data directly, again without user configuration required. The format of the data and storage configurations do not need to be known to the user; you do not need to know that the data is stored as Cloud-Optimised GeoTiff (COG), Zarr or any other format. DataPoint knows how to open them for you - this is currently supported for cloud-based formats like COG, Zarr and Kerchunk, but in future will be extended for other file types like NetCDF/HDF, CSV, etc. An example of how to extract an xarray-based data representation from a given STAC item (regardless of format) is below:

```
from ceda_datapoint import DataPointClient

client = DataPointClient()
search = client.search(collections=['cmip6'],max_items=10)

# The search can render a list of the available data assets given this search criteria.
xr_dataset = search.open_dataset('CMIP6.ScenarioMIP.THU.CIESM.ssp585.r1i1p1f1.Amon.rsus.gr.v20200806-reference_file')
```

In general, users are advised to collect cloud assets into a cluster, rather than opening a dataset directly from the search object.

## Single-Search Selections

The selections made via the pystac-based DataPoint search, are now applied directly to the data where possible. This minimises the extra configuration required to get to your specific spatial/temporal area of interest (AOI). The following search parameters are now applied directly to the data as standard:

- **intersects**: Search query for accessing STAC records within a specific AOI, this area will then be applied to the data produced when performing ``open_dataset`` so your data cube is representative of the search specified. (Note: This is supported for standard regular-grid coordinates only - namely lat/lon or variations of those. This is an experimental feature, please report any issues on the GitHub repo - link above)

- **datetime**: Search query for finding STAC records that fall within a datetime range. This range is then applied to the data cube/array on output. (Note: This is supported for the standard temporal dimension label ``time`` only. Arrays without a ``time`` dimension are not applicable. This is an experimental feature, please report any issues on the GitHub repo - link above) 

- **query.variables**: Pystac implements a metadata query parameter for searching specific fields in the STAC properties. For STAC records that contain a ``variables`` property, this search is applied directly to the data array on output, so your dataset contains just the variables you're searching for. This feature can also be utilised via the ``data_selections`` parameter specific to DataPoint - see below.

An example query where the single-search selections will be applied is shown here:

```
>>> client.search(
    collections=['example_collection'],
    intersects={
        "type": "Polygon",
        "coordinates": [[[6, 53], [7, 53], [7, 54], [6, 54], [6, 53]]],
    },
    datetime='2025-01-01/2025-12-31',
    query=[
        'cmip6:experiment_id=001',
        'variables=clt'
    ],
    data_selection={
        'variables':['clt','sst']
        'sel':{
            'nv':slice(0,5)
        }
    }
)
```

# Current CEDA STAC Collections

The CEDA STAC API server is still in an experimental stage, subject to change as the collections are updated and metadata is revised. As of the publication of this article there are 17 top-level collections via the main CEDA API, with CEDA-developed CCI collections being constructed in a separate location as part of the Knowledge Exchange program. For more details regarding CCI data, see the ESA Climate Office's [Open Data Portal](https://climate.esa.int/en/data/#/dashboard)

{% include figure.html
    image_url="assets/img/posts/2025-06-24-ceda-datapoint/STACBrowser.png"
    description="Figure 4: CEDA STAC Browser rendered by RadiantEarth"
%}

Some specific collections include cmip6 (CMIP Phase 6, provided by WCRP), cordex and various sentinel-based collections describing ARD products, which has been arranged in partnership with EOCIS. These STAC collections will be expanded over time, and are exposed for active use within EODH, detailed below.

## Earth Observation Data Hub

- Earth Observation Data Hub

# Next Steps

## Feedback from Living Planet: FAIR Data

The ESA Living Planet Symposium was hosted at the Austra Center Vienna conference centre from 23-27 June 2025. A presentation of DataPoint was given during the session entitled "Advancements in cloud-native formats and APIs for efficient management and processing of Earth Observation data" to an audience of data scientists, researchers and technical developers. The presentation will be published as part of the Living Planet proceedings publication and should be visible at [https://lps25.esa.int/](https://lps25.esa.int/).

Overall the response to the DataPoint package was positive, with some specific questions around long term support for the package and its use in scientific research. While DataPoint provides some very useful functionality for optimising data access and ease-of-use, none of the data processing is solely dependent on the use of the package against any other means. DataPoint simply takes advantage of the existing infrastructure within CEDA to provide better access to data, where other means exist but are more cumbersome and require greater manual configuration.

With that being said, the question of reusability and repeatability is something that will be taken into account in future DataPoint updates, specifically how a given set of data provided by a search can be represented in some way that can be preserved and maintained for future users. This should be made infrastructure-independent as far as possible, such that even if DataPoint and other dependencies are at some point decommissioned, the data is still clearly findable and accessible in the long term. In other words, considerations should be made such that DataPoint provides data under the FAIR data principles, which includes the reusability of both datasets and data workflows.

## Increased STAC Catalog Coverage

Works are ongoing within CEDA to expand the current STAC holdings that represent the archive to encompass more data. This includes the UK Climate Projections (UKCP) and Coupled Model Intercomparison Project (CMIP6/7), both of which have some limited STAC representation at present. DataPoint will continue to be promoted as a core package for use on JASMIN to access data in the CEDA Archive, and there is considerable desire to incorporate other projects and datasets into this model of data access.

# Getting in touch
If you would like to discuss this topic or other questions related to data Storage, Access and Discovery, please contact Daniel Westwood by [email](daniel.westwood@stfc.ac.uk).

# Further Resources
- [CEDA DataPoint Github](https://github.com/cedadev/datapoint)
- [DataPoint Documentation](https://cedadev.github.io/datapoint/)
- [CEDA STAC Browser](https://radiantearth.github.io/stac-browser/#/external/api.stac.ceda.ac.uk/?.language=en)
- [Earth Observation Data Hub](https://eodatahub.org.uk/)
- [STAC Specification](https://stacspec.org/en)
- [Pystac Client Documentation](https://pystac-client.readthedocs.io/en/stable/)
- [Kerchunk Documentation](https://fsspec.github.io/kerchunk/)
- [VirtualiZarr Documentation](https://virtualizarr.readthedocs.io/en/latest/)