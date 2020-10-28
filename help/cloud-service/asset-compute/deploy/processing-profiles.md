---
title: Integrate Asset Compute workers with AEM Processing Profiles
description: AEM as a Cloud Service integrates with Asset Compute workers deployed to Adobe I/O Runtime via AEM Assets Processing Profiles. Processing Profiles are configured in the Author service to process specific assets using custom workers, and store the files generated by the workers as asset renditions.
feature: asset-compute, processing-profiles
topics: renditions, development
version: cloud-service
activity: develop
audience: developer
doc-type: tutorial
kt: 6287
thumbnail: KT-6287.jpg
---

# Integrate with AEM Processing Profiles

For Asset Compute workers to generate custom renditions in AEM as a Cloud Service, they must be registered in AEM as a Cloud Service Author service via Processing Profiles. All assets subject to that Processing Profile will have the worker invoked upon upload or re-processing, and have the custom rendition generated and made available via the asset's renditions.

## Define a Processing Profile

First create a new Processing Profile that will invoke the worker with the configurable parameters.

![Processing profile](./assets/processing-profiles/new-processing-profile.png)

1. Login to AEM as a Cloud Service Author service as an __AEM Administrator__. As this is a tutorial we recommend using a Dev environment or an environment in a Sandbox.
1. Navigate to __Tools > Assets > Processing Profiles__
1. Tap __Create__ button
1. Name the Processing Profile, `WKND Asset Renditions`
1. Tap the __Custom__ tab, and tap __Add New__
1. Define the new service
    + __Rendition name:__ `Circle`
        + The file name rendition that will be used to identify this rendition in AEM Assets
    + __Extension:__ `png`
        + The extension of the rendition that will be generated. Set to `png` as this is the supported output format the worker's web service supports, and results in transparent background behind the circle cut out.
    + __Endpoint:__ `https://...adobeioruntime.net/api/v1/web/wkndAemAssetCompute-0.0.1/worker`
        + This is th URL to the worker obtained via `aio app get-url`. Ensure the URL points at the correct workspace based on the AEM as a Cloud Service environment the Processing Profile is being configured in. Note that this sub-domain matches the `development` workspace.
        + Make sure the worker URL points to the correct workspace. AEM as a Cloud Service Stage should use the Stage workspace URL, and AEM as a Cloud Service Production should use the Production workspace URL.
    + __Service Parameters__
        + Tap __Add Parameter__
            + Key: `size`
            + Value: `1000`
        + Tap __Add Parameter__
            + Key: `contrast`
            + Value: `0.25`
        + Tap __Add Parameter__
            + Key: `brightness`
            + Value: `0.10`
        + These key/value pairs that are passed into the Asset Compute worker and available via `rendition.instructions` JavaScript object.
    + __Mime Types__
        + __Includes:__ `image/jpeg`, `image/png`, `image/gif`, `image/bmp`, `image/tiff`
            + These MIME types are the only ones the worker's web service supports, this limits which assets can be processed by the custom worker.
        + __Excludes:__ `Leave blank`
            + Never process assets with these MIME Types using this service configuration. In this case, we only use an allow list.
1. Tap __Save__ in the top right

## Apply and invoke a Processing Profile

1. Select the newly created Processing Profile, `WKND Asset Renditions`
1. Tap __Apply Profile to Folder(s)__ in the top action bar
1. Select a folder to apply the Processing Profile to, such as `WKND` and tap __Apply__
1. Navigate to the folder the Processing Profile was not applied to via __AEM > Assets > Files__ and tap into `WKND`.
1. Upload some new images assets ([sample-1.jpg](../assets/samples/sample-1.jpg), [sample-2.jpg](../assets/samples/sample-2.jpg), and [sample-3.jpg](../assets/samples/sample-3.jpg)) in any folder under the folder with the Processing Profile applied, and wait for the uploaded asset to be processed.
1. Tap the asset to open its details
    + Default renditions may generate and appear more quickly in AEM than custom renditions.
1. Open the __Renditions__ view from the left sidebar
1. Tap on the asset named `Circle.png` and review the generated rendition

    ![Generated rendition](./assets/processing-profiles/rendition.png)

## Finished!

Congratulations! You have finished the [tutorial](../overview.md) on how to extend AEM as a Cloud Service Asset Compute microservices! You should now have the ability to set up, develop, test, debug and deploy custom Asset Compute workers for use by your AEM as a Cloud Service Author service.

### Review the full project source code on Github

The final Asset Compute project is available on Github at:

+ [aem-guides-wknd-asset-compute](https://github.com/adobe/aem-guides-wknd-asset-compute)

_Github contains is the final state of the project, fully populated with the worker and test cases, but does not contain any credentials, ie. `.env`, `.config.json` or `.aio`._

## Troubleshooting

### Custom rendition missing from asset

+ __Error:__ New and re-processed assets process successfully, but are missing the custom rendition

#### Processing profile not applied to ancestor folder

+ __Cause:__ The asset does not exist under a folder with the Processing Profile that uses the custom worker
+ __Resolution:__ Apply the Processing Profile to an ancestor folder of the asset

#### Processing profile superseded by lower Processing Profile

+ __Cause:__ The asset exists beneath a folder with the custom worker Processing Profile applied, however a different Processing Profile that does not use the customer worker has been applied between that folder and the asset.
+ __Resolution:__ Combine, or otherwise reconcile, the two Processing Profiles and remove the intermediate Processing Profile

### Asset Processing Failed

+ __Error:__ Asset Processing Failed badge displayed on asset
+ __Cause:__ An error occurred in the execution of the custom worker
+ __Resolution:__ Follow the instructions on [debugging Adobe I/O Runtime activations](../test-debug/debug.md#aio-app-logs) using `aio app logs`.