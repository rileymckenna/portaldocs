
<a name="resourcemenublade"></a>
# ResourceMenuBlade

<a name="resourcemenublade-overview"></a>
## Overview

The resource menu provides an easy-to-use implementation of a menu blade, as specified in [top-blades-menublade.md](top-blades-menublade.md), with various common Azure resource menu items readily available. It does this by providing an app-like container for a resource, with a navigation menu on the left. This navigation menu allows an extension to access all of the functionality of the resource, categorized into relevant groups.

**What is required to enable the resource menu?**

The resource menu requires that the extension perform the following.

1. [Opt in by using an AssetType](#opt-in-by-using-an-assettype)

1. [Provide an AssetViewModel on the AssetType](#provide-an-assetviewmodel-on-the-assettype)

1. [Add a getMenuConfig method](#add-a-getmenuconfig-method)

1. [Add menu items to the menu](#add-menu-items-to-the-menu)

1. [Add ResourceMenu titles](#add-resourcemenu-titles)

1. [Validate that the UX is responsive](#validate-that-the-ux-is-responsive)

For more information about migrating an extension from the settings blade to the resource menu, see
[portalfx-resourcemenu-migration.md](portalfx-resourcemenu-migration.md).

For more information about the APIs that support the Resource menu, see [portalfx-resourcemenu-api.md](portalfx-resourcemenu-api.md).

For answers to frequently asked questions, see [portalfx-extensions-faq-resourcemenu.md](portalfx-extensions-faq-resourcemenu.md).

If there are any issues please reach out to <a href="mailto:ibiza-menu-blade@microsoft.com?subject=Issues with Resource Samples">Ibiza Menu Blade</a>.

**What does the resource menu provide that a menu blade does not?**

The resource menu is a blade which is loaded from the HubsExtension. It  calls the `AssetViewModel` that is  associated with the `AssetType` to determine what menu items to show and what blades those items open. Every extension and resource can leverage this blade without worrying about getting UX consistency and avoid having to implement the same blade multiple times. The resource menu also provides an `options` object on the `getMenuConfig` which exposes various Azure resource-related options such as `Support/Monitoring/Automation Scripts.`  

<a name="resourcemenublade-overview-opt-in-by-using-an-assettype"></a>
### Opt in by using an AssetType

**NOTE**: In this discussion, `<dir>` is the `SamplesExtension\Extension\` directory, and  `<dirParent>` is the `SamplesExtension\` directory, based on where the samples were installed when the developer set up the SDK. If there is a working copy of the sample in the Dogfood environment, it is also included.

The code for this example is located at `<dir>Client\V1\Navigation\ResourceMenuBlade\ViewModels`.

<!-- TODO:  Determine whether this is the default experience yet. -->

For the extension to use a resource menu, it should provide `UseResourceMenu="true"` as a property on the `AssetType`  tag in the PDL file, as in the following code. 

```xml
<AssetType Name="MyResource"
           ...
           UseResourceMenu="true">
```

For this flag to take effect, the `assetType` should map to an ARM resource. The ARM resource is associated with the  `assetType` by specifying the following tags within the `<AssetType>` tag.

```xml
<Browse Type="ResourceType" />
<ResourceType
    ResourceTypeName="<<your-resource-type-name>>"
    ApiVersion="api-version-you-want-to-use" />
```

As an example, the resource type tag for resource groups is in the following code.

```xml
<ResourceType
    ResourceTypeName="Microsoft.Resources/subscriptions/resourceGroups"
    ApiVersion="2014-04-01-preview" />
```

<a name="resourcemenublade-overview-provide-an-assetviewmodel-on-the-assettype"></a>
### Provide an AssetViewModel on the AssetType

Create a new `ViewModel` for the `AssetViewModel`.  The following is a skeleton for the `AssetViewModel`. For more information on assets, see [portalfx-assets.md](portalfx-assets.md).


<!--
```ts
/**
 * The MyResource view model.
 */
export class MyResourceViewModel
    implements ExtensionDefinition.ViewModels.MyResourceViewModel.Contract {
 Also look at \Client\V1\ResourceTypes\Desktop\AssetViewModels
    /**
     * Initializes a new instance of the desktop view model class.
     *
     * @param container Object representing the container in the shell.
     * @param initialState Bag of properties saved to user settings via viewState.
     * @param dataContext Long lived data access object passed into all view models in the current area.
     */
    constructor(container: FxContainerContract, initialState: any, dataContext: DataContext) {
    }
}
 ```


-->

After the `AssetViewModel` has been added, there should also be a reference to it from the `AssetType` in the PDL file.

```xml
<AssetType Name="MyResource"
           ...
           ViewModel="{ViewModel Name=MyResourceViewModel, Module=./AssetViewModels/MyResourceViewModel}">
```

### Add a getMenuConfig method 

The next step in creating a ResourceMenu is to add a `getMenuConfig` method to the `AssetViewModel`.

The method is named `getMenuConfig` and uses the following signature.  For a full list of the APIs and interfaces, see [portalfx-resourcemenu-api.md](portalfx-resourcemenu-api.md).

```ts
public getMenuConfig(resourceInfo: MsPortalFx.Assets.ResourceInformation): MsPortalFx.Base.PromiseV<MsPortalFx.Assets.ResourceMenuConfig> {
    return Q({});
}
```

The following object is populated and sent to the `getMenuConfig` method. If you would like to see more properties added here, reach out to <a href="mailto:ibiza-menu-blade@microsoft.com?subject=Resource Sample Needs More Properties ">Ibiza Menu Blade</a>.

```ts
/**
 * The resource information for the resource menu.
 */
interface ResourceInformation {
    /**
     * The resource ID.
     */
    resourceId: string;
    /**
     * The resource or resource group.
     */
    resource: FxAzure.Subscription | FxHubsAzure.ResourceGroup | FxHubsAzure.Resource;
}

```

### Add menu items to the menu

After the resource menu skeleton is working, create the menu object to return in the `getMenuConfig` method, as in the following code.

```ts
/**
 * The resource menu configuration.
 */
interface ResourceMenuConfig {
    /**
     * The menu item groups.
     */
    groups: FxMenuBlade.MenuGroup[];
    /**
     * The ID of the default menu item.
     */
    defaultItemId: string;
    /**
     * Optional set of resource menu options.
     */
    options?: ResourceMenuOptions;
}
```

The following properties can be sent to the  `getMenuConfig` method.

| Property        | Description |
|-----------------|-------------|
| `groups`        | Array of groups and menu items within each group that will open a blade |
| `defaultItemId` | ID of the menu item (defined in `groups`) to be selected by default |
| `options`       | Flags to show/hide common menu items |

The following options are available on  the `getMenuConfig` method.

| Option                        | Purpose | Production-ready metrics | Enabled by default | 
|-------------------------------|-------------|--------------------|----------|
| `enableAlerts`                | Create, view, and update alert rules. | No  | No |
| `enableAppInsights`           | View Application Insights monitoring. | No  | No | 
| `enableDiagnostics`           | View monitoring diagnostics. | No  | No | 
| `enableExportTemplate`        |  Export a template of the resource group to automate redeployments. RPs must provide [template schemas](http://aka.ms/armschema) for this. Does not support classic resources. | Yes | Resources, resource groups |
| `enableLocks`                 | Lock resources to avoid accidental deletion and/or editing. | Yes | Resources, resource groups, subscriptions | 
| `enableLogAnalytics`          | View OMS workspace. | No  | No | 
| `enableLogSearch`             | Search logs. | No  | No | 
| `enableMetrics`               | View monitoring metrics. | No  | No | 
| `enableProperties`            | Generic properties blade for resources. Only includes standard ARM properties today, but may be integrated with the supplemental data, if needed. (Please file a [partner request](http://aka.ms/portalfx/request).) Does not support non-tracked resources. | No  | No | 
| `enableRbac`                  | Manage user/role assignments for this resource. | Yes | All ARM resource types | 
| `enableSupportEventLogs`      | View all operations and events | Yes | Resources, resource groups, subscriptions | 
| `enableSupportHelpRequest`    | Create a support request for this resource, resource group, or subscription. | Yes | All ARM resource types | 
| `enableSupportResourceHealth` | Check resource for common health issues (e.g. connectivity) and recommend fixes. |Yes | No | 
| `enableSupportTroubleshoot`   | **Deprecated. Do not use.** Legacy support only. Moved to a new design with improved usability scores. | No  | No | 
| `enableSupportTroubleshootV2` | Troubleshoot possible availability/reliability issues (e.g. connectivity). | Yes | No | 
| `enableTags`                  | Tag resource with key/value pairs to group/organize related resources. RP must support PATCH operations to update tags. Does not support classic resources. | Yes | Resources, resource groups, subscriptions | 
| `showAppInsightsFirst`        | View Application Insights monitoring. `enableAppInsights` must be set to `true`. | No  | No | 

In this example, the resource contains an item with the ID 'overview' and has also onboarded to extension Support, as specified in [top-blades-settings.md#audit-logs](top-blades-settings.md#audit-logs). This support automatically makes the export template, locks, RBAC, Activity Log, new support request, and tags available to the extension.

```ts
public getMenuConfig(resourceInfo: MsPortalFx.Assets.ResourceInformation): MsPortalFx.Base.PromiseV<MsPortalFx.Assets.ResourceMenuConfig> {
    return Q(
        <MsPortalFx.Assets.ResourceMenuConfig>{
            defaultItemId: "overview",
            options: {
                enableSupportTroubleshoot: true,
                enableSupportResourceHealth: true
            },
            groups: <FxMenuBlade.MenuGroup[]>[
                ...
            ]
        }
    );
}
```

To define a menu group that contains a single item, the menu's group and item API are as follows.

```ts
/**
 * Defines a group in the menu.
 */
interface MenuGroup extends MenuItemBase {
    /**
     * The menu items in the group.
     */
    items: MenuItem[];
}

/**
 * Defines an item in a group of the menu.
 */
interface MenuItem extends MenuItemBase, FxComposition.Selectable2Options<FxComposition.BladeReference<any>> {
    /**
     * The icon associated to the menu item.
     */
    icon: FxBase.Image;
    /**
     * A value indicating whether or not the item is enabled.
     */
    enabled?: KnockoutObservableBase<boolean>;
}

/**
 * Attributes common to all items and groups in the menu.
 */
interface MenuItemBase {
    /**
     * Gets the ID for the item.
     */
    id: string;
    /**
     * The display text for the item.
     */
    displayText: string;
    /**
     * A space-delimited list of keywords associated to the item.
     */
    keywords?: string | string[];
}
```

This means that the `getMenuConfig` method will resemble the following.

```ts
import * as ClientResources from "ClientResources";
import * as FxMenuBlade from "MsPortalFx/Composition/MenuBlade";
import * as BladeReferences from "../../../_generated/BladeReferences";

public getMenuConfig(resourceInfo: MsPortalFx.Assets.ResourceInformation): MsPortalFx.Base.PromiseV<MsPortalFx.Assets.ResourceMenuConfig> {
    return Q(
        <MsPortalFx.Assets.ResourceMenuConfig>{
            defaultItemId: "overview",
            options: {
                enableSupportTroubleshoot: true,
                enableSupportResourceHealth: true
            },
            groups: <FxMenuBlade.MenuGroup[]>[
                {
                    id: "overview_group",
                    displayText: ClientResources.ResourceMenuGroup.overview,
                    items: [
                        {
                            id: "overview",
                            displayText: ClientResources.ResourceMenu.overview,
                            enabled: ko.observable(true),
                            keywords: "overview",
                            icon: Images.MyResourceIcon,
                            supplyBladeReference: () => {
                                return new BladeReferences.MyResourceOverviewBlade({ id: resourceInfo.resourceId });
                            }
                        }
                    ]
                }
            ]
        }
    );
}
```

At this point, the extension has a Resource menu that contains one group, with one item in it, that  opens the `MyResourceOverviewBlade` blade.

### Add ResourceMenu titles

In this overview, the resource menu by default will show the name of the
resource as the title of the blade, and the type of resource as the subtitle. When
any menu item is selected, the title gets updated to `<<resource name>> - <<selected menu item>>`.

Developers can add to the title and subtitle through the blade that was opened in
the content area. They can do this by implementing the `HostedInMenuBlade`
interface as follows.

```ts
export class MyResourceBlade
    extends MsPortalFx.ViewModels.Blade
    implements MsPortalFx.ViewModels.HostedInMenuBlade {

    public menuContent = {
        title: ko.observable<string>(),
        subtitle: ko.observable<string>(),
    };

    public onInputsSet(inputs: any): MsPortalFx.Base.Promise {
        return someAsyncCall().then(data => {
            this.menuContent.title(data.title);
            this.menuContent.subtitle(data.subtitle);
        });
    }
}
```

<a name="resourcemenublade-overview-validate-that-the-ux-is-responsive"></a>
### Validate that the UX is responsive

Because the resource menu acts as a container for the blades opened by the menu items, the display state is preserved when the user switches between menu items. The developer should validate that blades render the UX properly when the resource menu is maximized, and also when the resource menu is restored to the specified widths.

For an example of responsive resources, see the `<dir>Client\V1\ResourceTypes\Desktop\` folder, specifically the `...AssetViewModels\DesktopViewModel.ts` file.


<a name="resourcemenublade-frequently-asked-questions"></a>
## Frequently asked questions

<!-- TODO:  FAQ Format is ###Link, ***title***, Description, Solution, 3 Asterisks -->

<a name="resourcemenublade-frequently-asked-questions-the-resource-menu"></a>
### The Resource menu

***What is the resource menu?***

SOLUTION:  The resource menu is a single location for all the resource's functionality. It reduces horizontal movement by promoting a consistent navigation model for all Azure resources.

* * *

<a name="resourcemenublade-frequently-asked-questions-resource-menu-samples"></a>
### Resource menu samples

***Are there any samples I can refer to?***

SOLUTION: There are numerous samples that demonstrate how to use the resource menu.  They are located at `..<dir>\Client\ResourceTypes\Desktop\AssetViewModels\DesktopViewModel.ts`, where `<dir>` is the `SamplesExtension\Extension\` directory, based on where the samples were installed when the developer set up the SDK. 

* * *

<a name="resourcemenublade-frequently-asked-questions-the-support-resource-management-group"></a>
### The Support Resource Management Group

***How do I add items to the Support/Resource Management Group?***

SOLUTION:  You can add items by using a `MenuGroupExtension`. `MenuGroupExtension` is a special kind of menu group that is  specified in the menu config object.  For more information, see 
[portalfx-resourcemenu-migration.md#creating-an-assetviewmodel](portalfx-resourcemenu-migration.md#creating-an-assetviewmodel).

* * * 

<a name="resourcemenublade-frequently-asked-questions-horizontal-scrolling"></a>
### Horizontal scrolling

***How do I reduce horizontal scrolling and UI movement in my extension?***

Horizontal scrolling and UI movement was a prime source of negative user feedback. One way of addressing this issue is to refactor the extension so that there are fewer blades on the user's journey. The average journey depth is three or four blades and the average flow is Browse, Resource blade, Settings and then some blade from Settings or Resource menu. In many cases, the fourth blade is displayed off screen and then scrolled into view.  Using a Resource blade reduces the dependency on the settings blade. In some instances, an extension no longer uses a Settings blade, thereby reducing the number of blades on the journey.

* * *
<!--
gitdown": "include-file", "file": "../templates/portalfx-extensions-glossary-resourcemenu.md"}
-->