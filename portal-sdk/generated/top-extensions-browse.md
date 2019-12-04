<a name="browse"></a>
# Browse

The Favorites in the left nav and the 'All services' menu are the primary ways to launch tools and services within the portal. The default favorites are determined by C+E leadership based on the highest grossing services with the most engaged customers. New services will start in the 'All services' menu and, based on those metrics or the number of favorites surpasses other defaults, the list can be updated.

<a name="browse-building-browse-experiences"></a>
## Building browse experiences

There are 2 ways you can be surfaced in Browse:

1. [Browse](#browse-for-arm-resources) for ARM resources

    - Upgrading your browse to utilise [Azure Resource Graph](#browse-with-azure-resource-graph)

1. [Custom blade](#custom-blade) if you have a single instance and not a list of resources

<a name="browse-for-arm-resources"></a>
# Browse for ARM resources

ARM Browse automatically queries ARM for resources of a specific type and displays them in a paged grid.

Simply;

1. Define an asset type in PDL
1. Specify the resource type
1. Indicate that it should be visible in Browse
1. Specify the API version that hubs extension should use to call ARM for the resource type.

That's it, you can see an example of that below

```xml
<AssetType Name="Book" ... >
  <Browse Type="ResourceType" />
  <ResourceType ResourceTypeName="Microsoft.Press/books" ApiVersion="2016-01-01" />
</AssetType>
```

![No-code Browse grid](../media/portalfx-browse/nocode.png)

All asset types have the following requirements:

1. The asset type blade **_must_** have a single `id` parameter that is the asset id
1. The asset type part must be the same as the blade's pinned part

> Asset types that represent Azure Resource Manager (ARM) resource types also have the following requirements:

1. The asset id **_must_** be the string resource id
1. The ARM RP manifest should include a RP, resource type, and resource kind metadata

<a name="browse-for-arm-resources-defining-your-asset-type"></a>
## Defining your asset type
To define your asset type, simply add the following snippet to PDL:

```xml
<AssetType
    Name="MyAsset"
    ServiceDisplayName="{Resource MyAsset.service, Module=ClientResources}"
    SingularDisplayName="{Resource MyAsset.singular, Module=ClientResources}"
    PluralDisplayName="{Resource MyAsset.plural, Module=ClientResources}"
    LowerSingularDisplayName="{Resource MyAsset.lowerSingular, Module=ClientResources}"
    LowerPluralDisplayName="{Resource MyAsset.lowerPlural, Module=ClientResources}"
    Keywords="{Resource MyAsset.keywords, Module=ClientResources}"
    Description="{Resource MyAsset.description, Module=ClientResources}"
    Icon="{Resource Content.MyExtension.Images.myAsset, Module=./../_generated/Svg}"
    BladeName="MyAssetBlade"
    PartName="MyAssetPart"
    IsPreview="true">
  ...
</AssetType>
```

The name can be anything, since it's scoped to your extension. You'll be typing this a lot, so keep it succinct, yet clear -- it will be used to identify asset types in telemetry.

In order to provide a modern voice and tone within the portal, asset types have 4 different display names. The portal will use the most appropriate display name given the context. If your asset type display name includes an acronym or product name that is always capitalized, use the same values for upper and lower display name properties (e.g. `PluralDisplayName` and `LowerPluralDisplayName` may both use `SQL databases`). Do not share strings between singular and plural display name properties.

* The 'All services' (browse) menu shows the `ServiceDisplayName` in the list of browseable asset types. If `ServiceDisplayName` is not available, `PluralDisplayName` will be shown instead
* The All Resources blade uses the `SingularDisplayName` in the Type column, when visible
* Browse v2 uses the `LowerPluralDisplayName` when there are no resources (e.g. "No web apps to display")
* Browse v2 uses the `LowerPluralDisplayName` as the text filter placeholder

Filtering functionality within the 'All services' (browse) menu searches over `Keywords`. `Keywords` is a comma-separated set of words or phrases which
allow users to search for your asset by identifiers other than than the set display names.

Remember, your part and blade should both have a single `id` input parameter, which is the asset id:

```xml
<Part Name="MyAssetPart" ViewModel="MyAssetPartViewModel" AssetType="MyAsset" AssetIdProperty="id" ...>
  <Part.Properties>
    <!-- Required. Must be the only input parameter. -->
    <Property Name="id" Source="{DataInput Property=id}" />
  </Part.Properties>
  <BladeAction Blade="MyAssetBlade">
    <BladeInput Source="id" Parameter="id" />
  </BladeAction>
  ...
</Part>

<Blade Name="MyAssetBlade" ViewModel="MyAssetBladeViewModel" AssetType="MyAsset" AssetIdProperty="id">
  <Blade.Parameters>
    <!-- Required. Must be the only input parameter. -->
    <Parameter Name="id" Type="Key" />
  </Blade.Parameters>
  <Blade.Properties>
    <Property Name="id" Source="{BladeParameter Name=id}" />
  </Blade.Properties>
  ...
</Blade>
```

If your asset type is in preview, set the `IsPreview="true"` property. If the asset type is GA, simply remove the property (the default is `false`).

<a name="browse-for-arm-resources-defining-your-asset-type-how-to-hide-or-show-your-asset-in-different-environments"></a>
### How to hide or show your asset in different environments

You can hide or show your asset in different environments by setting the assettypeoptions extension feature flag in your config which is a rich object
structure which allows changing not only asset types, but also hiding and showing asset instances in browse and global search as well as hiding and showing
asset instances with a specific resource kind. *This supersedes the legacy hideassettypes extension feature flag.*

```json
    {
        "assettypeoptions": {
          "YOUR_ASSET_NAME": { "options": "HideAssetType" },
          "YOUR_OTHER_ASSET_NAME": { "options": "HideAssetType,HideInstances" },
          "YOUR_THIRD_ASSET_NAME": { "options": "" }
        }
    }
```

The "options" value is a comma-separated list of options which will be applied to the asset type:

Options| Result
--- | ---
HideAssetType | Hides the asset type from the All services left navigation
HideInstances | Hides any instances of the asset type in browse and global search
HideAssetType,HideInstance | Hide the asset type from left navigation AND hides any instances in browse and global search
*empty string* | This will show the asset type in left navigation AND shows instances in browse and global search

The case of empty string strips off any visibility options provided in PDL. The options are applied to the asset type, essentially replacing the options in PDL.

As mentioned above, visibility of instances with a specific resource kind can also be controlled if the kind is specified in the PDL:

```json
    {
        "assettypeoptions": {
          "YOUR_ASSET_NAME": { "options": "HideAssetType" },
          "YOUR_OTHER_ASSET_NAME": { "options": "HideAssetType,HideInstances" },
          "YOUR_THIRD_ASSET_NAME": { "options": "" },
          "YOUR_ASSET_WITH_KINDS_NAME": { "kinds": { "KIND_NAME": { "options": "HideInstances" } } }
        }
    }
```

In this example, the instances of the asset type 'YOUR_ASSET_WITH_KINDS_NAME' which have the kind of 'KIND_NAME' will be hidden. Note that the 'HideAssetType' option does
not apply to kind, only the 'HideInstances' option.

There is also no way to show hidden asset types or hide / show specific kinds using the old configuration flags, so please move to 'assettypeoptions'.

<a name="browse-for-arm-resources-defining-your-asset-type-how-to-hide-or-show-your-asset-in-different-environments-self-hosted"></a>
#### Self hosted

This now reads the config JSON file for the appropriate environment, so follow the same procedure as for the Hosting service next.

<a name="browse-for-arm-resources-defining-your-asset-type-how-to-hide-or-show-your-asset-in-different-environments-hosting-service"></a>
#### Hosting service

If you’re using the hosting service, you can do this by updating your domainname.json (e.g. portal.azure.cn.json file)

```json
    {
        "assettypeoptions": {
          "YOUR_ASSET_NAME": { "options": "HideAssetType" },
          "YOUR_OTHER_ASSET_NAME": { "options": "HideAssetType,HideInstances" },
          "YOUR_THIRD_ASSET_NAME": { "options": "" },
          "YOUR_ASSET_WITH_KINDS_NAME": { "kinds": { "KIND_NAME": { "options": "HideInstances" } } }
        }
    }
```

**IMPORTANT** These flags cannot be mixed with the legacy 'hideassettypes' flag. If the config provides an 'assettypeoptions' flag, 'hideassettypes' flag
will be ignored. This is one reason that using 'assettypeoptions' is preferred and use of 'hideassettypes' should be deprecated.

<a name="browse-for-arm-resources-defining-your-asset-type-how-to-hide-or-show-your-asset-in-different-environments-testing-your-hidden-asset"></a>
#### Testing your hidden asset

To test enable your hidden asset for testing purposes, you will need to update the hide asset feature flag to exclude the asset you want to show and ensure you have feature.canmodifyextensions set.

For the desired environment append the following feature flags.
> If you want to test showing all hidden assets, you can specify all the assets as a comma separated list to the 'showassettypes' extension feature flag.

```txt
    ?microsoft_azure_mynewextension_assettypeoptions={"MyNewAsset":{"options":""},"MySecondNewAsset":{"options":""}}
```

For example:
https://rc.portal.azure.com/?microsoft_azure_compute_assettypes={"VirtualMachine":{"options":""}}&microsoft_azure_compute=true&feature.canmodifyextensions=true

or testing the hiding of an asset can be achieved with:
https://rc.portal.azure.com/?microsoft_azure_compute_assettypes={"VirtualMachine":{"options":"HideAssetType"}}&microsoft_azure_compute=true&feature.canmodifyextensions=true

<a name="browse-for-arm-resources-defining-your-asset-type-how-to-hide-or-show-your-asset-in-different-environments-how-the-options-are-applied-from-pdl-from-the-config-json-file-and-from-the-url"></a>
#### How the options are applied from PDL, from the config JSON file and from the URL

There is a definitive recipe for how visibility options are applied to asset types and kinds from the various sources of PDL, config JSON files and via
overrides on the URL. The PDL has the lowest priority and should be considered the defaults. Whatever options you apply in PDL will be already applied to
the asset type. Once the portal loads, the config JSON file for the appropriate domain is loaded and overrides are applied from there. The options flags
are replaced by the flags from config. The 'assettypeoptions' are applied if present, otherwise 'hideassettypes' are applied if present. Note that if
both are supplied, only the 'assettypeoptions' will be used. After that, the URL extension feature flag will have the highest priority and will be applied
last. Again, if the 'MyExtensionName_assettypeoptions' feature flag is in the URL, it will be applied.  Otherwise, if the 'MyExtensionName_showassettypes'
feature flag is present, it will be applied. Otherwise, if the 'MyExtensionName_hideassettypes' feature flag is present, it will be applied. Note that
the only asset types affected are those in the config or feature flag. Other asset types will not be affected:

1. PDL flags are baked into the asset type definition at compile time.
2. Config is applied:
3. If the 'assettypeoptions' is present, apply any changes, jump to step 5
4. If the 'hideassettypes' is present, apply any changes, jump to step 5 - this is considered legacy and should be replaced with assettypeoptions.
5. URL overrides are applied:
6. If the 'MyExtensionName_assettypeoptions' feature flag is present, apply any changes, jump to end
7. If the 'MyExtensionName_showassettypes' feature flag is present, apply any changes, jump to end - this is considered legacy and should be replaced with assettypeoptions.
8. If the 'MyExtensionName_hideassettypes' feature flag is present, apply any changes, jump to end - this is considered legacy and should be replaced with assettypeoptions.

As shown, if 'assettypeoptions' and 'hideassettypes' are all present in the config, the 'hideassettypes' will be ignored. The 'hideassettypes' flag is considered legacy and
should be replaced with 'assettypeoptions'.

Also, if 'assettypeoptions', 'showassettypes' and 'hideassettypes' are all present in the URL, the 'showassettypes' and 'hideassettypes' will be ignored
and if only 'showassettypes' and 'hideassettypes' are specified, 'hideassettypes' will be ignored. Both 'showassettypes' and 'hideassettypes' are both considered to be legacy
and should be replaced with 'assettypeoptions'.

<a name="browse-for-arm-resources-defining-your-asset-type-handling-arm-kinds"></a>
### Handling ARM kinds

If the resource you wish to expose does not have kinds then please skip to the next topic.

ARM has the capability for a resource to define kinds, in some cases you may want to
treat those kinds separately in the portal.

To define a kind for your asset, you need to declare the kind as a child of the `Asset` within PDL.
Firstly you will need to specify a default kind, this kind inherits the blade/part defined in the Asset.
The default kind is identified with `IsDefault="true"`.

If your resource exposes multiple kinds you can declare them as siblings of the default kind.

Exposing your kind within the 'All services' menu will require your kind/asset to be curated within the Portal Framework. The framework also offers ways for grouping kinds together when browsing to those kinds.
There are two options you can use group your kinds:

1. KindGroup
    * This will define a separate `KindGroup` within your extensions definition which can be used as a way to define a single view for mulitple kinds while also keeping the individual kind view.
1. MergedKind
    * Similar to `KindGroup` `MergedKind`s will group various kinds together and present them in a single view, except `MergedKind` forces any instance of the individual kinds to be viewed as the merged view.

```xml
<!--
    This asset type represents a watch instance.

    An asset type represents an asset object in the system independent of other objects in the system.
    It represents a singular class of objects distinctively but without connection to other objects.

    This asset type includes a resource type which represents a watch instance in the resource map.

    A resource type represents an asset specifically in a resource map where the connections between
    objects is important.  It represents a way to map resources in a resource map to the underlying
    assets in the system.

    It includes the resource map icons which are used in the resource map control.

    Watch is an "abstract" asset type, there is no such thing as a "watch", the default
    watch type is a "apple" watch.  Other specializations are based on function.
  -->
  <AssetType Name="Watch"
             ViewModel="{ViewModel Name=WatchViewModel, Module=./Watch/AssetType/Watch}"
             CompositeDisplayName="{Resource AssetTypeNames.Watch, Module=ClientResources}"
             Icon="{Svg IsLogo=true, File=../../Svg/Watches/generic.svg}"
             BladeName="AppleWatchBlade"
             PartName="AppleWatchTile"
             UseResourceMenu="true"
             MarketplaceItemId="Microsoft/watch">
    <Browse Type="ResourceType"
            UseCustomConfig="true"
            UseSupplementalData="true" />
    <ResourceType ResourceTypeName="Microsoft.Test/watches"
                  ApiVersion="2017-04-01">
      <Kind Name="apple"
            IsDefault="true"
            CompositeDisplayName="{Resource AssetTypeNames.Watch.Apple, Module=ClientResources}"
            Icon="{Svg IsLogo=true, File=../../Svg/Watches/apple.svg}" />
      <Kind Name="lg"
            CompositeDisplayName="{Resource AssetTypeNames.Watch.LG, Module=ClientResources}"
            Icon="{Svg IsLogo=true, File=../../Svg/Watches/lg.svg}"
            BladeName="LgWatchBlade"
            PartName="LgWatchTile" />
      <Kind Name="samsung"
            CompositeDisplayName="{Resource AssetTypeNames.Watch.Samsung, Module=ClientResources}"
            Icon="{Svg IsLogo=true, File=../../Svg/Watches/samsung.svg}"
            BladeName="SamsungWatchBlade"
            PartName="SamsungWatchTile" />
      <Kind Name="fitbit"
            CompositeDisplayName="{Resource AssetTypeNames.Watch.Fitbit, Module=ClientResources}"
            Icon="{Svg IsLogo=true, File=../../Svg/Watches/fitbit.svg}"
            BladeName="FitbitWatchBlade"
            PartName="FitbitWatchTile" />
      <!--
        The 'android' kind group wraps the lg and samsung kinds into a single kind. The 'android' kind is an abstract
        kind. There should never be a watch with the kind set to 'android'. Instead it's used to group kinds into
        a single list. However, 'lg' watches and be seen separately, same with 'samsung' watches. The 'android' kind
        will be emitted to the manifest as a kind.
      -->
      <KindGroup Name="android"
            CompositeDisplayName="{Resource AssetTypeNames.Watch.Android, Module=ClientResources}"
            Icon="{Svg IsLogo=true, File=../../Svg/Watches/android.svg}">
        <KindReference KindName="lg" />
        <KindReference KindName="samsung" />
      </KindGroup>
      <!--
        The 'garmin-merged' kind has two merged kinds, 'garmin' and 'garmin2'. The 'garmin-merged' kind is not a real
        kind and is not emitted to the manifest as a kind, it is organizational only.
      -->
      <MergedKind Name="garmin-merged">
        <Kind Name="garmin"
              CompositeDisplayName="{Resource AssetTypeNames.Watch.Garmin, Module=ClientResources}"
              Icon="{Svg IsLogo=true, File=../../Svg/Watches/garmin.svg}"
              BladeName="GarminWatchBlade"
              PartName="GarminWatchTile" />
        <Kind Name="garmin2"
              CompositeDisplayName="{Resource AssetTypeNames.Watch.Garmin2, Module=ClientResources}"
              Icon="{Svg IsLogo=true, File=../../Svg/Watches/garmin2.svg}"
              BladeName="Garmin2WatchBlade"
              PartName="Garmin2WatchTile" />
      </MergedKind>
    </ResourceType>
  </AssetType>
```

<a name="browse-for-arm-resources-defining-your-asset-type-add-command"></a>
### Add command

To allow people to create new resources from Browse, you can associate your asset type with a Marketplace item or category:

```xml
<AssetType
    Name="Book"
    MarketplaceItemId="Microsoft.Book"
    MarketplaceMenuItemId="menu/submenu"
    ...>
  <Browse Type="ResourceType" />
  <ResourceType ResourceTypeName="Microsoft.Press/books" ApiVersion="2016-01-01" />
</AssetType>
```

The Browse blade will launch the Marketplace item, if specified; otherwise, it will launch the Marketplace category blade for the specific menu item id (e.g. `gallery/virtualMachines/recommended` for Virtual machines > Recommended). To determine the right Marketplace category, contact the <a href="mailto:1store?subject=Marketplace menu item id">Marketplace team</a>. If neither is specified, the Add command won't be available.

<a name="browse-for-arm-resources-defining-your-asset-type-handling-empty-browse"></a>
### Handling empty browse

The framework offers the ability to display a description and links in the case that the users filters return no results.

>NOTE: This will also display if the user visits the browse experience and they have not yet created the given resource.

![Empty browse](../media/portalfx-assets/empty-browse.png)

To opt in to this experience you need to provide a `description` and a `link`, these are properties that you provide on your Asset.

```xml
<AssetType
    Name="MyAsset"
    ...
    Description="{Resource MyAsset.description, Module=ClientResources}">
    ...
    <Link Title="{Resource MyAsset..linkTitle1, Module=ClientResources}" Uri="http://www.bing.com"/>
    <Link Title="{Resource MyAsset.linkTitle2, Module=ClientResources}" Uri="http://www.bing.com"/>
    ...
  </AssetType>
```

<a name="browse-for-arm-resources-defining-your-asset-type-customizing-columns"></a>
### Customizing columns

By default, ARM Browse only shows the resource name, group, location, and subscription. To customize the columns, add a view-model to the `AssetType` and indicate that you have custom Browse config:

```xml
<AssetType Name="Book" ViewModel="BookViewModel" ... >
  <Browse Type="ResourceType" UseCustomConfig="true" />
  <ResourceType ResourceTypeName="Microsoft.Press/books" ApiVersion="2016-01-01" />
</AssetType>
```

Now, create the asset view-model class that implements the `getBrowseConfig()` function:

```ts
class BookViewModel implements ExtensionDefinition.ViewModels.ResourceTypes.BookViewModel.Contract {

    public getBrowseConfig(): PromiseV<MsPortalFx.Assets.BrowseConfig> {
        ...
    }
}
```

The `getBrowseConfig()` function provides the following configuration options for your Browse blade:

* **`columns`** - List of custom columns the user will be able to choose to display
* **`defaultColumns`** - List of column ids that will be used by default
* **`properties`** - Additional properties used by formatted columns (e.g. HTML formatting)

Start by specifying all possible custom columns you want to make available to customers using `BrowseConfig.columns`. Browse will share the list of standard ARM columns and any custom columns you define with users and let them choose which columns they want to see.

To specify which columns to show by default, save the column ids to `BrowseConfig.defaultColumns`. If any columns require additional data, like HTML-formatted columns that include 2 or more properties, save the additional property names (not the `itemKey`) to `BrowseConfig.properties`. Browse needs to initialize the grid with all the properties you'll use for supplemental data to ensure the grid will be updated properly.

```ts
class BookViewModel implements ExtensionDefinition.ViewModels.ResourceTypes.BookViewModel.Contract {

    public getBrowseConfig(): PromiseV<MsPortalFx.Assets.BrowseConfig> {
        return Q.resolve({
            // columns the user will be able to choose to display
            columns: [
                {
                    id: "author",
                    name: ko.observable<string>(ClientResources.author),
                    itemKey: "author"
                },
                {
                    id: "genre",
                    name: ko.observable<string>(ClientResources.genre),
                    itemKey: "genre",
                    format: MsPortalFx.ViewModels.Controls.Lists.Grid.Format.HtmlBindings,
                    formatOptions: {
                        htmlBindingsTemplate: "<div data-bind='text: genre'></div> (<div data-bind='text: subgenre'></div>)"
                    }
                }
            ],

            // default columns to show -- name is always first
            defaultColumns: [
                ResourceColumns.resourceGroup,
                "author",
                "genre"
            ],

            // additional properties used to support the available columns
            properties: [
                "subgenre"
            ]
        });
    }
}
```

Notice that the genre column actually renders 2 properties: genre and subgenre. Because of this, we need to add "subgenre" to the array of additional properties to ensure it gets rendered properly to the grid.

At this point, you should be able to compile and see your columns show up in your Browse blade. Of course, you still need to populate your supplemental data. Let's do that now...

<a name="browse-for-arm-resources-defining-your-asset-type-providing-supplemental-data"></a>
### Providing supplemental data

In order to specify supplemental data to display on top of the standard resource columns, you'll need to opt in to specifying supplemental data in PDL:

```xml
<AssetType Name="Book" ViewModel="BookViewModel" ... >
  <Browse Type="ResourceType" UseSupplementalData="true" />
  <ResourceType ResourceTypeName="Microsoft.Press/books" ApiVersion="2016-01-01" />
</AssetType>
```

You'll also need to implement the `supplementalDataStream` property and `getSupplementalData()` function on your asset view-model:

```ts
class BookViewModel implements ExtensionDefinition.ViewModels.ResourceTypes.BookViewModel.Contract {

    public supplementalDataStream = ko.observableArray<MsPortalFx.Assets.SupplementalData>([]);

    public getBrowseConfig(): PromiseV<MsPortalFx.Assets.BrowseConfig> {
        ...
    }

    public getSupplementalData(resourceIds: string[], columns: string[]): Promise {
        ...
    }
}
```

After the Browse blade retrieves the first page of resources from ARM, it will call `getSupplementalData()` with the batch of resource ids retrieved from ARM as well as the column ids currently being displayed in the grid. You'll then retrieve *only* the properties required to populate those columns for *only* the specified resource ids. **Do not query all properties for all resources!**

```ts
class BookViewModel implements ExtensionDefinition.ViewModels.ResourceTypes.BookViewModel.Contract {

    private _container: MsPortalFx.ViewModels.ContainerContract;
    private _dataContext: any;
    private _view: any;

    constructor(container: MsPortalFx.ViewModels.ContainerContract, initialState: any, dataContext: ResourceTypesArea.DataContext) {
        this._container = container;
        this._dataContext = dataContext;
    }

    ...

    public getSupplementalData(resourceIds: string[], columns: string[]): Promise {
        // NOTE: Do not create the view in the constructor. Initialize here to create only when needed.
        this._view = this._view || this._dataContext.bookQuery.createView(this._container);

        // connect the view to the supplemental data stream
        MsPortalFx.Assets.SupplementalDataStreamHelper.ConnectView(
            this._container,
            view,
            this.supplementalDataStream,
            (book: Book) => {
                return resourceIds.some((resourceId) => {
                    return ResourceTypesService.compareResourceId(resourceId, book.id());
                });
            },
            (book: Book) => {
                // save the resource id so Browse knows which row to update
                var data = <MsPortalFx.Assets.SupplementalData>{ resourceId: book.id() };

                // only save author if column is visible
                if (columns.indexOf("author") !== -1) {
                    data.author = robot.author();
                }

                // if the genre column is visible, also add the subgenre property
                if (columns.indexOf("genre") !== -1) {
                    data.genre = robot.genre;
                    data.subgenre = robot.subgenre;
                }

                return data;
            });

        // send resource ids to a controller and aggregate data into one client request
        return view.fetch({ resourceIds: resourceIds });
    }
}
```

**NOTE:** If you notice that some of the supplemental properties aren't being saved to the grid, double-check that the property names are either listed as the `itemKey` for a column or have been specified in `BrowseConfig.properties`. Unknown properties won't be saved to the grid.

**NOTE:** **Do not pre-initialize data.** Browse will show a loading indicator based on whether or not it's received data. If you initialize any supplemental data, this will inform the grid that loading has completed. Instead, leave cells empty when first displaying them.

Now, you should have supplemental data getting populated. Great! Let's add context menu commands...

<a name="browse-for-arm-resources-adding-context-menu-commands"></a>
## Adding context menu commands

Context menu commands in Browse must take a single `id` input parameter that is the resource id of the specific resource. To specify commands, add the name of the command group defined in PDL to Browse config:

```xml
<CommandGroup Name="BrowseBookCommands">
  ...
</CommandGroup>
```

```ts
class BookViewModel implements ExtensionDefinition.ViewModels.ResourceTypes.BookViewModel.Contract {

    public getBrowseConfig(): PromiseV<MsPortalFx.Assets.BrowseConfig> {
        return Q.resolve({
            // NOTE: Extension (commandGroupOwner) only required if from another extension
            contextMenu: {
                commandGroup: "BrowseBookCommands",
                commandGroupOwner: "<extension name>"
            },
            ...
        });
    }

    ...
}
```

If you need to expose different commands based on some other metadata, you can also specify the the command group in `SupplementalData.contextMenu` in the same way.

<a name="browse-for-arm-resources-adding-context-menu-commands-adding-an-informational-message-link"></a>
### Adding an informational message/link

If you need to display an informational message and/or link above the list of resources, add an `infoBox` to your Browse config:

```ts
class BookViewModel implements ExtensionDefinition.ViewModels.ResourceTypes.BookViewModel.Contract {

    public getBrowseConfig(): PromiseV<MsPortalFx.Assets.BrowseConfig> {
        return Q.resolve({
            infoBox: {
                image: MsPortalFx.Base.Images.Info(),
                text: resx.browseBookInfoBoxText,

                // optionally specify a blade to launch when the infobox is clicked
                blade: <MsPortalFx.ViewModels.DynamicBladeSelection>{
                    detailBlade: "BookInfoBlade",
                    detailBladeInputs: null
                },

                // ...or link to an external web page
                uri: "http://microsoftpress.com"

                // NOTE: Blade is preferred over link, if both are specified.
           },
            ...
        });
    }

    ...
}
```
<a name="browse-with-azure-resource-graph"></a>
# Browse with Azure Resource Graph

If you aren’t familiar Azure Resource Graph, it’s a new service which provides a query-able caching layer over ARM.
This gives us the capability to sort, filter, and search server side which is a vast improvement on what we have today.

What that does mean though is we won’t be loading extensions to gather the extra, supplemental, column data. Instead that will all be served via ARG.

Due to which there the following required from extension authors to onboard.

- Define the columns which you wish to expose
- Craft the query to power your data set
  - To craft the query you can use the in portal advanced query editor [Azure Resource Graph Explorer](https://rc.portal.azure.com#blade/hubsextension/argqueryblade)
  - Ensure the query projects all the framework and extension expected columns
- Onboard your given asset via PDL
  - If you haven't created an Asset follow the previous documentation on how to do that
- Choose how to expose the ARG experience

**Note:** the below contains the PDL, Columns definitions, and Query required to match to an existing AppServices browse experience.

<a name="browse-with-azure-resource-graph-onboarding-an-asset-to-arg"></a>
## Onboarding an asset to ARG

Firstly you'll need to craft a KQL query which represents all possible data for your desired browse view, this includes the required framework columns.

<a name="browse-with-azure-resource-graph-onboarding-an-asset-to-arg-expected-framework-columns"></a>
### Expected Framework columns

| Display name | Expected Column Name | PDL Reference |
| ------------ | -------------------- | ------------- |
| Name | name | N/A - Injected as the first column |
| Resource Id | id | FxColumns.ResourceId |
| Subscription | N/A | FxColumns.Subscription |
| SubscriptionId | subscriptionId | FxColumns.SubscriptionId |
| Resource Group | resourceGroup | FxColumns.ResourceGroup |
| Resource Group Id | N/A | FxColumns.ResourceGroupId |
| Location | location | FxColumns.Location |
| Location Id | N/A | FxColumns.LocationId |
| Resource Type | N/A | FxColumns.ResourceType |
| Type | type | FxColumns.AssetType |
| Kind | kind | FxColumns.Kind |
| Tags | tags | FxColumns.Tags |
| Tenant Id | tenantId | N/A |

<a name="browse-with-azure-resource-graph-kql-query"></a>
## KQL Query

For those who are not familar with KQL you can use the public documentation as reference. https://docs.microsoft.com/en-us/azure/kusto/query/

Given the framework columns are required we can use the below as a starting point.

1. Go to the [Azure Resource Graph explorer](https://rc.portal.azure.com#blade/hubsextension/argqueryblade)
1. Copy and paste the below query
1. Update the `where` filter to your desire type

```kql
where type =~ 'microsoft.web/sites'
| project name,resourceGroup,kind,location,id,type,subscriptionId,tags
```

That query is the bare minimum required to populate ARG browse.

As you decide to expose more columns you can do so by using the logic available via the KQL language to `extend` and then `project` them in the query.
One common ask is to convert ARM property values to user friendly display strings, the best practice to do that is to use the `case` statement in combination
with extending the resulting property to a given column name.

In the below example we're using a `case` statement to rename the `state` property to a user friendly display string under the column `status`.
We're then including that column in our final project statement. We can then replace those display strings with client references once we migrate it over to
PDL in our extension providing localised display strings.

```kql
where type =~ 'microsoft.web/sites'
| extend state = tolower(properties.state)
| extend status = case(
state == 'stopped',
'Stopped',
state == 'running',
'Running',
'Other')
| project name,resourceGroup,kind,location,id,type,subscriptionId,tags
, status
```

As an example the below query can be used to replicate the 'App Services' ARM based browse experience in ARG.

```kql
where type =~ 'microsoft.web/sites'
| extend appServicePlan = extract('serverfarms/([^/]+)', 1, tostring(properties.serverFarmId))
| extend appServicePlanId = properties.serverFarmId
| extend state = tolower(properties.state)
| extend sku = tolower(properties.sku)
| extend pricingTier = case(
sku == 'free',
'Free',
sku == 'shared',
'Shared',
sku == 'dynamic',
'Dynamic',
sku == 'isolated',
'Isolated',
sku == 'premiumv2',
'PremiumV2',
sku == 'premium',
'Premium',
'Standard')
| extend status = case(
state == 'stopped',
'Stopped',
state == 'running',
'Running',
'Other')
| extend appType = case(
kind contains 'botapp',
'Bot Service',
kind contains 'api',
'Api App',
kind contains 'functionapp',
'Function App',
'Web App')
| project name,resourceGroup,kind,location,id,type,subscriptionId,tags
, appServicePlanId, pricingTier, status, appType
```

<a name="browse-with-azure-resource-graph-pdl-definition"></a>
## PDL Definition

In your extension you'll have a `<Asset>` tag declared in PDL which represents your ARM resource. In order to enable Azure Resource Graph (ARG) support for that asset we'll need to update the `<Browse>` tag to include a reference to the `Query`, `DefaultColumns`, and custom column meta data - if you have any.

<a name="browse-with-azure-resource-graph-pdl-definition-query-for-pdl"></a>
### Query for PDL

Create a new file, we'll use `AppServiceQuery.kml`, and save your query in it.
You can update any display strings with references to resource files using following syntax `'{{Resource name, Module=ClientResources}}'`.

The following is an example using the resource reference syntax.

```kql
where type == 'microsoft.web/sites'
| extend appServicePlanId = properties.serverFarmId
| extend state = tolower(properties.state)
| extend sku = tolower(properties.sku)
| extend pricingTier = case(
    sku == 'free',
    '{{Resource pricingTier.free, Module=BrowseResources}}',
    sku == 'shared',
    '{{Resource pricingTier.shared, Module=BrowseResources}}',
    sku == 'dynamic',
    '{{Resource pricingTier.dynamic, Module=BrowseResources}}',
    sku == 'isolated',
    '{{Resource pricingTier.isolated, Module=BrowseResources}}',
    sku == 'premiumv2',
    '{{Resource pricingTier.premiumv2, Module=BrowseResources}}',
    sku == 'premium',
    '{{Resource pricingTier.premium, Module=BrowseResources}}',
    '{{Resource pricingTier.standard, Module=BrowseResources}}')
| extend status = case(
    state == 'stopped',
    '{{Resource status.stopped, Module=BrowseResources}}',
    state == 'running',
    '{{Resource status.running, Module=BrowseResources}}',
    '{{Resource status.other, Module=BrowseResources}}')
| extend appType = case(
    kind contains 'botapp',
    '{{Resource appType.botapp, Module=BrowseResources}}',
    kind contains 'api',
    '{{Resource appType.api, Module=BrowseResources}}',
    kind contains 'functionapp',
    '{{Resource appType.functionapp, Module=BrowseResources}}',
    '{{Resource appType.webapp, Module=BrowseResources}}')
| project name,resourceGroup,kind,location,id,type,subscriptionId,tags
, appServicePlanId, pricingTier, status, appType
```

<a name="browse-with-azure-resource-graph-pdl-definition-custom-columns"></a>
### Custom columns

To define a custom column you will need to create a `Column` tag in PDL within your `Browse` tag.

A column tag has 5 properties.

```xml
<Column Name="status"
      DisplayName="{Resource Columns.status, Module=ClientResources}"
      Description="{Resource Columns.statusDescription, Module=ClientResources}"
      Format="String"
      WidthInPixels="90" />
```

- Name: The identifier which is used to uniquely refer to your column
- DisplayName: A display string, __this has to be a reference to a resource__
- Description: A description string, __this has to be a reference to a resource__
- Format: See below table for possible format
- WidthInPixels: String, which respresents the default width of the column in pixels (e.g. "120")

| Format option | Description |
| ------------- | ----------- |
| String | String rendering of your column |
| Resource | If the returned column is a ARM resource id, this column format will render the cell as the resources name and a link to the respective blade |

<a name="browse-with-azure-resource-graph-pdl-definition-default-columns"></a>
### Default columns

To specify default columns you need to declare a property `DefaultColumns` on your `Browse` `PDL` tag.
Default columns is a comma separated list of column names, a mix of custom columns and framework defined columns from the earlier table. All framework columns are prefixed with `FxColumns.`.

For example `DefaultColumns="status, appType, appServicePlanId, FxColumns.Location"`.

<a name="browse-with-azure-resource-graph-pdl-definition-full-asset-definition"></a>
### Full Asset definition

In the above query example there are 4 custom columns, the below Asset `PDL` declares the custom column meta data which each map to a column in the query above.

It also declares the default columns and their ordering for what a new user of the browse experience should see.

```xml
<AssetType>
    <Browse
        Query="{Query File=./AppServiceQuery.kml}"
        DefaultColumns="status, appType, appServicePlanId, FxColumns.Location">
            <Column Name="status"
                  DisplayName="{Resource Columns.status, Module=ClientResources}"
                  Description="{Resource Columns.statusDescription, Module=ClientResources}"
                  Format="String"
                  WidthInPixels="90" />
            <Column Name="appType"
                  DisplayName="{Resource Columns.appType, Module=ClientResources}"
                  Description="{Resource Columns.appTypeDescription, Module=ClientResources}"
                  Format="String"
                  WidthInPixels="90" />
            <Column Name="appServicePlanId"
                  DisplayName="{Resource Columns.appServicePlanId, Module=ClientResources}"
                  Description="{Resource Columns.appServicePlanIdDescription, Module=ClientResources}"
                  Format="Resource"
                  WidthInPixels="90" />
            <Column Name="pricingTier"
                  DisplayName="{Resource Columns.pricingTier, Module=ClientResources}"
                  Description="{Resource Columns.pricingTierDescription, Module=ClientResources}"
                  Format="String"
                  WidthInPixels="90" />
    </Browse>
</AssetType>
```

<a name="browse-with-azure-resource-graph-releasing-the-azure-resource-graph-experience"></a>
## Releasing the Azure Resource Graph experience

Ensure the SDK version that you use is `5.0.302.27001` or later.

Per Asset you can configure extension side feature flags to control the release of your assets Azure Resource Graph browse experience.

Within your extension config, either hosting service or self hosted, you will need to specify config for your assets with one of the following:

```json
{
    "argbrowseoptions": {
        "YOUR_ASSET_NAME": "OPTION_FROM_THE_TABLE_BELOW",
    }
}
```

| Option | Definition |
| ------ | ---------- |
| AllowOptIn | Allows users to opt in/out of the new experience but will default to the old experience. This will show a 'Try preview' button on the old browse blade and an 'Opt out of preview' button on the ARG browse blade. |
| ForceOptIn | Allows users to opt in/out of the new experience but will default to the new experience. This will show a 'Try preview' button on the old browse blade and an 'Opt out of preview' button on the ARG browse blade |
| Force | This will force users to the new experience. There wil be no 'Opt out of preview' button on the ARG browse blade |
| Disable | This will force users to the old experience. This is the default experience if not flags are set. There wil be no 'Try preview' button on the ARG browse blade |

To test each variation or to test when side loading you can use:

`https://portal.azure.com/?ExtensionName_argbrowseoptions={"assetName":"OPTION"}`

<a name="custom-blade"></a>
# Custom blade

If you don't have a list of resources and simply need to add a custom blade to Browse, you can define an asset type with a `Browse` type of `AssetTypeBlade`. This tells Browse to launch the blade associated with the asset type. Note that the asset type doesn't actually refer to an instance of a resource in this case. This is most common for services that are only provisioned once per directory or horizontal services (Cost Management, Monitoring, Azure Advisor etc...). In this case, the `PluralDisplayName` is used in the 'All services' menu, but the other display names are ignored. Feel free to set them to the same value.

```xml
<AssetType
    Name="CompanyLibrary"
    BladeName="CompanyLibraryBlade"
    PluralDisplayName="..."
    ... >
  <Browse Type="AssetTypeBlade" />
</AssetType>
```

<a name="curating-browse-assets"></a>
# Curating browse assets

When adding a new 'Asset' to your extension and exposing it through the 'All services' menu, by default it will be grouped in the 'Other' category.
In order to get your 'Asset' correctly categorized you will need to make a request to the Portal Framework to curate your 'Asset'.

For the portal to correct curate your 'Asset' we will need the 'ExtensionName' and 'AssetName' as defined in your extension.

Please contact [ibizafxpm@microsoft.com](mailto:ibizafxpm@microsoft.com) with the following template:

Subject: 'Browse curation requestion - YourAssetName'
Body:

- 'ExtensioName - YourExtensionName'
- 'AssetName - YourAssetName'
- 'KindName - YourKindName' (If applicable)
- 'Category - DesiredCategory'
- 'Portal environment - portal.azure.com (etc...)'
