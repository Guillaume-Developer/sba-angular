---
layout: default
title: Network Module - BETA
parent: Components
grand_parent: Modules
nav_order: 20
---

# Network Module - BETA

## Reference documentation

Please checkout the [reference documentation]({{site.baseurl}}components/modules/NetworkModule.html) auto-generated from source code.

Also checkout the official documentation of the [Vis.js network](https://visjs.github.io/vis-network/docs/network/) library and its [Github repository](https://github.com/visjs/vis-network).

## Features

Sinequa is **not** a graph database, such as [Neo4J](https://neo4j.com/) or [Janus Graph](https://janusgraph.org/). However, data stored in Sinequa indexes can be visualized and explored in a graphical way. The graph structure is not defined at the database level (and it is therefore impossible to compute complex queries involving graph traversal), it is defined at the User Interface level, where data fetched from the server dynamically can be formatted and assembled into a graph.

This module includes a component which displays a network view (based on the [Vis.js network](https://visjs.github.io/vis-network/docs/network/) library). This view displays nodes and edges generated by one or several *providers* ([`NetworkProvider`]({{site.baseurl}}components/interfaces/NetworkProvider.html)). Each provider generates a *dataset* ([`NetworkDataset`]({{site.baseurl}}components/interfaces/NetworkDataset.html)) of nodes and edges, which are then merged into a single one before displaying the network.

![Network]({{site.baseurl}}assets/modules/network/network.png){: .d-block .mx-auto }

## Import

Import this module in your `app.module.ts`.

```ts
import { NetworkModule } from '@sinequa/components/network';

@NgModule({
  imports: [
    ...
    NetworkModule
```

For some functionalities of the module, you will also need to import the following stylesheet in your app’s global stylesheet:

```scss
// Vis.js styles
@import "~vis-network/dist/vis-network.min.css";
```

This module is internationalized: If not already the case, you need to import its messages for the language(s) of your application. For example, in your app's `src/locales/en.ts`:

```ts
...
import {enNetwork} from "@sinequa/components/network";

const messages = Utils.merge({}, ..., enNetwork, appMessages);
```

## Quick start

If you are in a hurry, and just want to try out the network with a configuration that works regardless of the specifics of your project, try the following:

1. Import the `NetworkModule` in your project, as shown above.
2. In the template of one of your components, insert the `sq-network` component:

    ```html
    <sq-facet-card [title]="'Network'" [icon]="'fas fa-project-diagram'">
        <sq-network #facet [results]="results" [providers]="providers"></sq-network>
    </sq-facet-card>
    ```

3. In the controller of that component, import an Out-of-the-box sample and import the [`ProviderFactory`]({{site.baseurl}}components/injectables/ProviderFactory.html). Then initialize the `providers` variable:

    ```ts
    import { NetworkProvider, ProviderFactory, oOTBConfig } from '@sinequa/components/network';

    @Component({
      ...
    })
    export class MyComponent {

      providers: NetworkProvider[] = [];

      constructor(
        ...,
        public providerFactory: ProviderFactory
      ) {
        this.providers = oOTBConfig(providerFactory);
      }
    ```

At this point, you should have a network component displaying standard entities (people, companies, geos), and their relations with selected documents. Other samples are available (most of them documented below). To explore them, look in the `network-sample-config.ts` file.

A natural next step is to customize these samples by copying and modifying their code into your component.

At some point, you may feel restricted by what is shown in these samples. We recommend looking into the code of the factory itself (`provider-factory.ts`), to learn how to further customize node types, edge types and providers.

Finally, keep in mind that nothing prevents you from programming your own providers. Have a look at the existing ones, and try using inheritance when possible (at the very least, extending the [`BaseProvider`]({{site.baseurl}}components/class/BaseProvider.html)).

## Network Component

The [`sq-network`]({{site.baseurl}}components/components/NetworkComponent.html) component is a facet component (See [Facet module](facet.html)), which is best used within a facet card:

```html
<sq-facet-card [title]="'Network'" [icon]="'fas fa-project-diagram'">
    <sq-network #facet [results]="results" [providers]="providers"></sq-network>
</sq-facet-card>
```

The component primarily requires a list of **providers** as input ([`NetworkProvider`]({{site.baseurl}}components/interfaces/NetworkProvider.html) objects), which generate the data displayed in the network. The `results` input is essentially used to refresh the providers when new results come in.

## Architecture

The goal of the network component is to display data in the form of **nodes** ([`Node`]({{site.baseurl}}components/interfaces/Node.html) objects) and **edges** ([`Edge`]({{site.baseurl}}components/interfaces/Edge.html) objects).

These node and edge objects are generated by **providers** ([`NetworkProvider`]({{site.baseurl}}components/interfaces/NetworkProvider.html) objects). The [`sq-network`]({{site.baseurl}}components/components/NetworkComponent.html) component constantly listens to its input providers, and combines all their nodes and edges into a single graph.

Nodes and edges are bundled in a container called a **dataset** ([`NetworkDataset`]({{site.baseurl}}components/class/NetworkDataset.html) class). So providers essentially emit a dataset (whenever the data needs to be refreshed), and the component just *merges* the latest version of the datasets emitted by each provider.

![Architecture]({{site.baseurl}}assets/modules/network/architecture.png){: .d-block .mx-auto }

The purpose of this architecture is to break down the vast space of network configurations into small composable bricks. For example, it is very easy to combine data coming from aggregations with data coming from records.

## Interface

The [`NetworkProvider`]({{site.baseurl}}components/interfaces/NetworkProvider.html) interface contains the following fields and methods:

- `active: boolean`: Whether or not the provider is active (if inactive, it will provide empty datasets of nodes and edges)
- `name: string`: A name for this provider (for displays in menus)
- `dataset: NetworkDataset`: A dataset object which is filled by this provider
- `context: NetworkContext`: A convenience wrapper containing some globally accessible parameters
- `getProvider(): Subject<NetworkDataset>`: Returns the Subject of this provider. The subject is an event emitter which can provide a new dataset at any time.
- `getData(context: NetworkContext)`: Typically called on initialization, or when new results come in, to trigger a new Dataset to be emitted by the provider (synchronously or not).
- `onDatasetsMerged(dataset: NetworkDataset)`: A method called after the datasets provided by all providers have been merged into a single dataset
- `onNodesInserted(nodes: Node[])`: A method called after the dataset is filtered (to keep only visible nodes) and passed to Vis for rendering
- `onNodeClicked(node: Node | undefined)`: A method called when ANY node is cliked in the rendered view of the network
- `onEdgeClicked(edge: Edge | undefined)`: A method called when ANY edge is cliked in the rendered view of the network
- `getProviderActions(): Action[]`: A method called to retrieve the list of action for this provider
- `getNodeActions(node: Node): Action[]`: A method called to retrieve the list of actions for a given node, and this provider.
- `getEdgeActions(edge: Edge): Action[]`: A method called to retrieve the list of actions for a given edge, and this provider.
- `onDestroy()`: A method called when the providers are discarded. Can be use to cancel subscriptions.

This interface is implemented by a number of pre-packaged providers documented below. Developers are of course welcome to develop their own providers to customize how the network component works. Note that a [`BaseProvider`]({{site.baseurl}}components/classes/BaseProvider.html) is available, with convenience methods to create nodes and edges, and a default implementation of the [`NetworkProvider`]({{site.baseurl}}components/interfaces/NetworkProvider.html) interface.

## Node and Edge types

Providers require as inputs some *types* for the nodes and edges. These types are respectively [`NodeType`]({{site.baseurl}}components/interfaces/NodeType.html) and [`EdgeType`]({{site.baseurl}}components/interfaces/EdgeType.html) objects. These types determine the visual appearances (color, size, etc.) of the nodes and edges, thanks to a wide range of options available in the [Vis.js](https://visjs.github.io/vis-network/docs/network/) library.

The [`NodeType`]({{site.baseurl}}components/interfaces/NodeType.html) interface contains the following fields and options:

- `name: string`: An identifier for this node type
- `nodeOptions`: Node options define the appearance of a node. The full list of available node options is available in the [Vis.js documentation](https://visjs.github.io/vis-network/docs/network/nodes.html). `nodeOptions` can be a static object (all the nodes with this type will look the same) or a function returning an object (which allows to customize the appearance for each node).
- `field?: string`: An optional field name for this node type. If provided, the node will have the ability to filter the search (for example clicking on the node "Paris", will let the user filter the search with a selection of the form `geo:=Paris`).

**Examples:**

A node type with static options:

```ts
const type = {
  name: "geo",
  nodeOptions: {
    shape: "icon",
    icon: {
      face: "'FontAwesome'",
      code: "\uf007",
      size: 50,
      color: "#aa00ff",
    },
  }
}
```

A node type with dynamic options (the `size` is determined in function of the node's label):

```ts
const type = {
  name: "geo",
  nodeOptions: (node: Node, type: NodeType) => {
    const size = node.label === "Paris"? 100 : 50;
    return {
      shape: "icon",
      icon: {
        face: "'FontAwesome'",
        code: "\uf007",
        size: size,
        color: "#aa00ff",
      },
    }
  }
}
```

The [`EdgeType`]({{site.baseurl}}components/interfaces/EdgeType.html) interface contains the following fields and options:

- `nodeTypes: NodeType[]`: The node types for each side of the edge. Normally two node types must be provided (except in some special cases).
- `edgeOptions`: Edge options define the appearance of an edge. The full list of available edge options is available in the [Vis.js documentation](https://visjs.github.io/vis-network/docs/network/edges.html). `edgeOptions` can be a static object (all the edges with this type will look the same) or a function returning an object (which allows to customize the appearance for each edge).
- `field?: string`: An optional field name for this edge type. If provided, the edge will have the ability to filter the search (the provider generating this edge must include the `fieldValue` for the edge). Alternatively, an edge can let the user filter both its adjacent nodes (if both of them have a field).

## Provider Factory

Rather than creating providers and node types manually, the [`ProviderFactory`]({{site.baseurl}}components/injectables/ProviderFactory.html) can be injected into an app in order to easily generate the objects and configuration needed in a specific project:

```ts
import { ProviderFactory } from '@sinequa/components/network';

@Component({
  ...
})
export class MyComponent {

  constructor(
    ...
    public providerFactory: ProviderFactory
  )
```

The factory includes convenience methods for creating all built-in types of nodes, edges and providers.

## Samples

On top of the factory, we provide samples of provider configurations (generated via the factory) in `network-sample-config.ts`.

![Factory]({{site.baseurl}}assets/modules/network/factory.png){: .d-block .mx-auto }

The following providers are all illustrated by multiple samples. Samples are convenient to quickly start displaying data, but they must be modified for any customized project. We recommand copying the code of the sample(s) closest to your needs into your project, and modifying it from there.

## List of providers

### Records Provider

The class [`RecordsProvider`]({{site.baseurl}}components/classes/RecordsProvider.html) provides nodes that are generated from a list of [`Record`]({{site.baseurl}}components/interfaces/Record.html) objects ("record" is a generic term for the content of a Sinequa index, returned in the form of `Record` objects by the [query web service]({{site.baseurl}}core/injectables/QueryWebService.html)).

Additionally, the provider can generate nodes and edges for the **properties** contained in that record. For example, if a record has "person" entities as a property, each of these "person" entity can be displayed as a property linked to the record:

```ts
// Create a standard "document" node type
const doc = providerFactory.createRecordNodeType();
// Create a standard "person" node type
const person = providerFactory.createPersonNodeType();

// Create an edge type for the "person" property of the "document" node
const struct = providerFactory.createStructuralEdgeTypes(doc, [person], "oninsert", "all");

// Create a provider that provides document nodes with their structural neighbors, given a list of record objects
const provider = providerFactory.createRecordsProvider(doc, struct, records);
```

![Record provider]({{site.baseurl}}assets/modules/network/records-provider.png){: .d-block .mx-auto }

The sample code above demonstrate the usage of the factory's `createStructuralEdgeTypes()` method. Additional methods exist to create advanced types of structural edges:

- `createCoocStructuralEdgeTypes()` creates edges for cooccurrence entities (proximity relations).
- `createTypedCoocStructuralEdgeTypes()` creates edges for typed cooccurrence entities (semantic relations).

For example:

```ts
const doc = providerFactory.createRecordNodeType();
const person = providerFactory.createPersonNodeType();
const company = providerFactory.createCompanyNodeType();

const struct = providerFactory.createTypedCoocStructuralEdgeTypes(doc, [person, company], "person_job_company", "oninsert", "all");

const provider = providerFactory.createRecordsProvider(doc, [struct], records);
```

![Typed cooc record provider]({{site.baseurl}}assets/modules/network/record-typed-cooc-base.png){: .d-block .mx-auto }

Note that it is possible to hide the underlying record node, if we are only interested in the metadata:

```ts
// Note the fourth argument (true)
const provider = providerFactory.createRecordsProvider(doc, [struct], records, true);
```

![Typed cooc record provider]({{site.baseurl}}assets/modules/network/record-typed-cooc.png){: .d-block .mx-auto }

### Selected Records Providers

The [`SelectedRecordsProvider`]({{site.baseurl}}components/classes/SelectedRecordsProvider.html) class is a direct extension of [`RecordsProvider`]({{site.baseurl}}components/classes/RecordsProvider.html). The difference is that the provider listens to the [`SelectionService`]({{site.baseurl}}components/injectables/SelectionService.html) (See [Selection module](selection.html)) and provides record nodes from the list of selected records.

⚠️ The `SelectionService` must be set-up to store *records* instead of just *record ids* (See [Selection module](selection.html#selection-service)).

This provider lets you easily see the common properties of two or more selected records.

```ts
// Create a standard "document" node type
const doc = providerFactory.createRecordNodeType();
// Create a standard "person" node type
const person = providerFactory.createPersonNodeType();

// Create an edge type for the "person" property of the "document" node
const struct = providerFactory.createStructuralEdgeTypes(doc, [person], "oninsert", "all");

// Create a provider that provides document nodes with their structural neighbors, from the list of selected records
const provider = providerFactory.createSelectedRecordsProvider(doc, struct);
```

![Selected Record provider]({{site.baseurl}}assets/modules/network/selected-records-provider.png){: .d-block .mx-auto }

### Async Records Providers

The [`AsyncRecordsProvider`]({{site.baseurl}}components/classes/AsyncRecordsProvider.html) class is a direct extension of [`RecordsProvider`]({{site.baseurl}}components/classes/RecordsProvider.html). The difference is that the provider requires a [`Query`]({{site.baseurl}}core/classes/Query.html) object, which is used to fetch a list of records, and these records are then transformed into nodes (as described above).

```ts
// Create a custom "document" node type that displays an image (which URL is stored in the `sourcevarchar4` column)
const doc = providerFactory.createNodeType("doc",
    providerFactory.createDynamicImageNodeOptions(
        (node: Node) => (node as RecordNode).record['sourcevarchar4']
    )
);

// Build a query to retrieve articles about humans from a wikipedia index
const query = searchService.makeQuery();
query.text = "google";
query.addSelect("treepath:=/Web/Wiki/");
query.addSelect("sourcestr4:=human");
query.pageSize = 5;

// Create a provider that provides document nodes obtained from a query
const provider = providerFactory.createAsyncRecordsProvider(doc, [], query);
```

![Async Record provider]({{site.baseurl}}assets/modules/network/async-records-provider.png){: .d-block .mx-auto }

Notice here that we did not use the factory's built-in method for generating a node type for records/documents (`createRecordNodeType()`). Instead we used the generic `createNodeType()` method, and used the factory to generate **dynamic node options**. The options are "dynamic", because each node will look different from the others (indeed, we want each node to show the picture of a person). The `createDynamicImageNodeOptions()` requires a function taking the node as an input, and returning the URL of the image (in this case stored in the record's `sourcevarchar4` column). See [**Node and Edge types**](#node-and-edge-types).

### Aggregation Provider

The [`AggregationProvider`]({{site.baseurl}}components/classes/AggregationProvider.html) class provides nodes and edges generated from an *aggregation*. Aggregations are computed by the Sinequa engine based on the content of one column (or more) of an index. Aggregation are typically used to compute the content of facets (See [Facet Module](facet.html)).

The aggregation provider can be used to generate different types of relations between metadata:

- **Statistical relations**: In this case, we use **cross-aggregations** to count the number of records which contain two values of metadata. For example, we can count how many documents both have `docformat=pdf` and `author=John Doe`. The most frequent pairs of metadata are translated into edges between these metadata.
- **Proximity relations**: In this case, we use aggregations to count the number of **cooccurence entities** stored within a specific column. A cooccurrence entity stores the occurrence of two other entities within a short range of text, like in the sentence *"**Larry Page** works at **Google**"*, which could be normalized as `(LARRY PAGE)#(GOOGLE)`. The cooccurrences are then translated as edges between each entity.
- **Semantic relations**: In this case, we use aggregations to count the number of **semantic entities** stored within a specific column. A semantic entity stores the typed relation between two entities, like in the sentence *"**Larry Page** is an engineer who **works at** the company **Google**"* (unlike in the previous example, "work at" can be normalized as a type of relation between the two entities, so that this sentence could be stored as `(GOOGLE)#(EMPLOYS)#(LARRY PAGE)`). Then this relation between the two entities can be visualized as a directed typed edge.

Assuming the entities are properly extracted from documents, normalized and stored in columns of the index, the following example describe how to use the [`AggregationProvider`]({{site.baseurl}}components/classes/AggregationProvider.html) to produce different types of network visualizations.

Note that in all cases, **aggregations are computed in the context of the current query of the [`SearchService`]({{site.baseurl}}components/injectables/SearchService.html)**, unless a different `Query` is passed to the provider as an optional input. This means if you search for "Google", the top relations built by this provider should normally include companies like Google, and people like Larry Page.

#### **Statistical relations**

First we need to configure the cross-aggregation calculation on the server, in the **Query web service** (see [Server-side setup]({{site.baseurl}}gettingstarted/server-setup.html#apps)).

![Cross distribution configuration]({{site.baseurl}}assets/modules/network/cross-dist.png){: .d-block .mx-auto }
*Configuration for a cross-aggregation between the **company** and **person** columns*
{: .text-center }

```ts
// Create two standard node types for persons and companies
const person = providerFactory.createPersonNodeType();
const company = providerFactory.createCompanyNodeType();

// Create an edge type between these two node types
const edge = providerFactory.createAggregationEdgeType([company, person], "Company_Person")

// Create the provider, given the edge type
const provider = providerFactory.createAggregationProvider([edge]);
```

![Cross distribution provider]({{site.baseurl}}assets/modules/network/cross-dist-provider.png){: .d-block .mx-auto }

Note that the size of each node is proportional to the width of adjacent edges, which itself reflects the "count" of each pair in the distribution (which is equal to the number of documents containing both metadata).

#### **Proximity relations**

First we need to configure the aggregation calculation on the server, in the **Query web service** (see [Server-side setup]({{site.baseurl}}gettingstarted/server-setup.html#apps)).

The aggregation must be computed for a column where cooccurrences are stored in the format `(VALUE 1)#(VALUE 2)`.

The code below is almost identical to the one above for statistical relations. The difference is that we are calling the `createCoocAggregationEdgeType()` method instead of `createAggregationEdgeType()`.

```ts
// Create two standard node types for persons and companies
const person = providerFactory.createPersonNodeType();
const company = providerFactory.createCompanyNodeType();

// Create an cooccurrence aggregation edge type between these two node types
const edge = providerFactory.createCoocAggregationEdgeType([company, person], "Company_Person_Cooc")

// Create the provider, given the edge type
const provider = providerFactory.createAggregationProvider([edge]);
```

![Cooccurrence provider]({{site.baseurl}}assets/modules/network/cooccurrences.png){: .d-block .mx-auto }

#### **Semantic relations**

First we need to configure the aggregation calculation on the server, in the **Query web service** (see [Server-side setup]({{site.baseurl}}gettingstarted/server-setup.html#apps)).

The aggregation must be computed for a column where cooccurrences are stored in the format `(VALUE 1)#(TYPE)#(VALUE 2)`.

The code below is almost identical to the one above for statistical relations. The difference is that we are calling the `createTypedCoocAggregationEdgeType()` method instead of `createAggregationEdgeType()`.

```ts
// Create two standard node types for persons and companies
const person = providerFactory.createPersonNodeType();
const company = providerFactory.createCompanyNodeType();

// Create a typed cooccurrence aggregation edge type between these two node types
const edge = providerFactory.createTypedCoocAggregationEdgeType([person, company], "Person_Job_Company_Cooc")

// Create the provider, given the edge type
const provider = providerFactory.createAggregationProvider([edge]);
```

![Semantic relations]({{site.baseurl}}assets/modules/network/semantic-relations.png){: .d-block .mx-auto }

#### Expanding nodes

In the example aboves, the aggregation edges are created with `trigger="source"`, which means the data is fetched right away and inserted in the network.

But, it is also possible to use aggregations to attach new nodes to existing nodes ("expanding" them). In this case, instead of using cross-distributions or cooccurrence, a regular 1-dimension aggregation must be provided.

The provider will query the server for this distribution, and adding a WHERE clause to "fix" the source node. For example, if we want to expand the node `Paris` of type `geo`, with say, companies, the distribution queried to the server will be computed with:

```sql
SELECT DISTRIBUTION('company') as companies FROM ... WHERE ... AND geo = 'Paris' ...
```

Note that the `SKIP` and `COUNT` parameter of the distribution are automatically adjusted by the provider, so that expanding the node multiple times will yield more data (until exhaustion).

This computation is equivalent to a cross distribution of `geo` and `company` (where one half is fixed). Therefore combining "source" cross-distribution edges with "onclick" or "manual" will be consistent.

```ts
// Create two standard node types for persons and companies
const person = providerFactory.createPersonNodeType();
const company = providerFactory.createCompanyNodeType();

// Create an edge type between these two node types
const edge = providerFactory.createAggregationEdgeType([company, person], "Company_Person")

// Create an edge type to expand "company" with the "person" distribution. "manual" means a button is displayed when clicking on a company node to create the expansion.
const expandCompany = providerFactory.createAggregationEdgeType([company, person], "person", undefined, "manual");

// Create the provider, given the edge type
const provider = providerFactory.createAggregationProvider([edge, expandCompany]);
```

![Expand feature]({{site.baseurl}}assets/modules/network/expand.png){: .d-block .mx-auto }

### Dynamic providers

Dynamic providers are extensions of [`RecordsProvider`]({{site.baseurl}}components/classes/RecordsProvider.html). The list of records is fetched dynamically from the server, given a query. There are two types of dynamic providers:

- **Dynamic edge provider**: Provides record nodes (and optionally their structural edges) that will be attached to existing nodes via "dynamic edges".
- **Dynamic node provider**: Transforms an existing node into a record node (and its structural edges)

⚠️ Dynamic providers can potentially generate multiple parallel queries to the engine, depending on the settings you choose. Be careful not overflowing the engine and optimizing your queries as much as possible.

#### **Dynamic edge provider**

A dynamic edge provider ([`DynamicEdgeProvider`]({{site.baseurl}}components/classes/DynamicEdgeProvider.html)) attaches record nodes to an existing node. For example, we can create a first record provider that generates nodes for documents, and when these nodes are inserted, we query the engine for people nodes related to these documents.

```ts
// Create a standard document node type
const doc = providerFactory.createRecordNodeType();

// Create a standard provider for records
const recordProvider = providerFactory.createRecordsProvider(doc, [], records);

// Create a node type of people, displaying the image of that person instead of a generic icon
const people = providerFactory.createNodeType("person",
  providerFactory.createDynamicImageNodeOptions(
    (node: Node) => (node as RecordNode).record['sourcevarchar4'] || ""
  )
);

// Create a dynamic edge type, that creates a query to fetch people related to that document
const dynamicEdgeType = providerFactory.createDynamicEdgeType([doc, people], "oninsert",
  (node: Node, type: DynamicEdgeType) => {
    const query = searchService.makeQuery();
    query.text = node.label;
    query.addSelect("treepath:=/Web/Wiki/");
    query.addSelect("sourcestr4:=human");
    query.pageSize = 5;
    return query;
  });

// Create a dynamic edge provider to create the dynamic edges whose type we just defined
const peopleProvider = providerFactory.createDynamicEdgeProvider(dynamicEdgeType, [], true, "People", [recordProvider]);
```

![dynamic edge provider]({{site.baseurl}}assets/modules/network/dynamic-edges.png){: .d-block .mx-auto }

Note that the dynamic edge provider creates record nodes which can themselves have structural edges:

```ts
const company = providerFactory.createCompanyNodeType();
const struct = providerFactory.createStructuralEdgeTypes(people, [company]);
const peopleProvider = providerFactory.createDynamicEdgeProvider(dynamicEdgeType, struct, true, [recordProvider]);
```

![dynamic edge provider]({{site.baseurl}}assets/modules/network/dynamic-edges-struct.png){: .d-block .mx-auto }

#### **Dynamic node provider**

A dynamic node provider ([`DynamicNodeProvider`]({{site.baseurl}}components/classes/DynamicNodeProvider.html)) transforms ("mutates") an existing node into a record node. For example, a metadata node for the person "Bill Gates" can be enriched with the wikipedia article about Bill Gates.

In the following example, we start by creating an aggregation provider, displaying people and company relations. But the second provider enriches people nodes (when clicked on) with their wikipedia page (which allows to transform the appearance of the node and display their wikipedia pictures instead of an icon).

```ts
// Create the node types for the company and person entities
const company = providerFactory.createCompanyNodeType();
const person = providerFactory.makeNodeTypeDynamic(
  // By default, the node is a standard person node
  providerFactory.createPersonNodeType(),
  // This function returns the query necessary to transform the node
  (node: Node) => {
    let query = searchService.makeQuery();
    query.text = node.label;
    query.addSelect("treepath:=/Web/Wiki/");
    query.addSelect("sourcestr4:=human");
    query.pageSize = 1;
    return query
  },
  // The node options to use after the node has been transformed (displaying an image instead of an icon)
  providerFactory.createDynamicImageNodeOptions(
    (node: Node) => (node as RecordNode).record['sourcevarchar4'] || ""
  )
);

// Create structural edges from the document nodes to the person and company entities
const structEdges = providerFactory.createStructuralEdgeTypes(person, [company, person], "oninsert", "paginate");

// Create aggregation edges to link companies and people
const company_person = providerFactory.createAggregationEdgeType([company, person], "Company_Person");

// Create the aggregation provider
const aggProvider = providerFactory.createAggregationProvider([company_person]);

// Create the dynamic node provider
const personProvider = providerFactory.createDynamicNodeProvider(person, structEdges, false, "People", [aggProvider]);
```

![dynamic nodes provider]({{site.baseurl}}assets/modules/network/dynamic-nodes.png){: .d-block .mx-auto }

## Interactions

When clicking on a node or edge, it is possible to display **info cards**, which display information, and **actions**, which display buttons or menus associated with the given node or edge.

### Info cards

Info cards templates can be provided to the `sq-network` component by transclusion:

{% raw %}

```html
<sq-network #facet [results]="results" [providers]="providers">

    <ng-template #nodeTpl let-node>
        <h1>This node is named: {{ node.label }}</h1>
    </ng-template>

</sq-network>
```

{% endraw %}

And similarly for the edge info card:

{% raw %}

```html
<sq-network #facet [results]="results" [providers]="providers">

    <ng-template #edgeTpl let-edge>
        <h1>This edge goes from {{ edge.from }} to {{ edge.to }}</h1>
    </ng-template>

</sq-network>
```

{% endraw %}

Two components are provided as samples for node and edge info cards, but these should be typically modified and adapted to the specifics of the project.

Node and edge info cards:

```html
<sq-network #facet [results]="results" [providers]="providers">

    <ng-template #nodeTpl let-node>
        <sq-node-info-card [node]="node"></sq-node-info-card>
    </ng-template>

    <ng-template #edgeTpl let-edge>
        <sq-edge-info-card [edge]="edge"></sq-edge-info-card>
    </ng-template>

</sq-network>
```

![Node info card]({{site.baseurl}}assets/modules/network/node-info-card.png){: .d-block .mx-auto }

![Edge info card]({{site.baseurl}}assets/modules/network/edge-info-card.png){: .d-block .mx-auto }

### Actions

Actions associated to a node, edge or provider are displayed as buttons or menus in the facet card header:

![Actions]({{site.baseurl}}assets/modules/network/actions.png){: .d-block .mx-auto }

These "actions" (see [Action module](action.html)) come from the providers ([`NetworkProvider`]({{site.baseurl}}components/interfaces/NetworkProvider.html)) and can be customized by overriding an existing provider or implementing your own provider from scratch (in fact a provider can be created for the sole purpose of displaying actions for some categories of nodes or edges).

The providers have three methods that can be implemented to provide actions:

- `getProviderActions(): Action[]`: Return a list of [`Action`]({{site.baseurl}}components/classes/Action.html) objects associated to the **provider**. These actions are displayed regardless of the node or edge currently focused. The actions are nested in a menu displaying the actions for all providers:

![Actions for providers]({{site.baseurl}}assets/modules/network/actions-providers.png){: .d-block .mx-auto }
*List of actions for the provider "Aggregations"*
{: .text-center }

- `getNodeActions(node: Node): Action[]`: Returns a list of [`Action`]({{site.baseurl}}components/classes/Action.html) objects specific to a given **node**. These actions are displayed when the node is clicked or focused. ⚠️ This method is called when *ANY* node is clicked, not just nodes from this provider.

![Actions for providers]({{site.baseurl}}assets/modules/network/actions-node.png){: .d-block .mx-auto }
*List of actions for the node "Google"*
{: .text-center }

- `getEdgeActions(edge: Edge): Action[]`: Returns a list of [`Action`]({{site.baseurl}}components/classes/Action.html) objects specific to a given **edge**. These actions are displayed when the edge is clicked or focused. ⚠️ This method is called when *ANY* edge is clicked, not just edges from this provider.

Note that the [`BaseProvider`]({{site.baseurl}}components/class/BaseProvider.html) implementation already provides basic actions common to all providers, and all the providers documented above also provide actions when possible and relevant.

Below is a sample implementation (from the [`DynamicNodeProvider`]({{site.baseurl}}components/classes/DynamicNodeProvider.html) class) to display an action specific to a given node. Notice that we call the parent method (`super.getNodeActions(node)`) to also display the built-in actions from the parent. Also note that we *preprend* the list (with `unshift()`) to display this custom action *before* the built-in actions.

```ts
/**
 * Creates an action to process a clicked node, for dynamic node types
 * with a "manual" trigger.
 * @param node The clicked node
 */
getNodeActions(node: RecordNode): Action[] {
    const actions = super.getNodeActions(node);
    if(this.active && this.nodeType.trigger === "manual" && node && node.type === this.nodeType && this.processedNodes.indexOf(node.id) === -1) {
        actions.unshift(new Action({
            icon: "fas fa-star-of-life",
            title: this.context.intlService.formatMessage("msg#network.actions.expandNode", {label: node.label}),
            action: () => {
                this.processNode(node);
            }
        }));
    }
    return actions;
}
```