# The Project-Metadataschema: Descriptions and DC/EN-Mapping

The metadata schema developed for the project is based on core elements of the [Dublin Core (DC)](https://www.dublincore.org/) standard and [EN 15744](https://filmstandards.org/fsc/index.php/EN_15744). Key identification fields such as `title`, `identifier`, `country of production`, `year`, `director`, and `running time` align with established metadata elements. These fields serve both to ensure formal classification and clear identification, and to support further corpus analysis, for example by enabling calculations of country distribution or production periods.

The following table lists all `14 fields` of the corpus metadata schema along with their equivalents in Dublin Core (DC) and EN 15744. Fields without a mapping are project-specific extensional categories that have no direct equivalent in either standard (see `modes_intervention`).

For further details on patterns, standards, notes and examples, please refer to the [`yml`-schema](../schema/corpus_metadata_schema_climate_film.yml).

| Field | Description | Dublin Core | EN 15744 |
|---|---|---|---|
| `title` | Original title of the audiovisual resource in its original language | [dc:title](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#title) | Title |
| `object_id` | Project-specific unique identifier (ID) for the object (audiovisual resource) | [dc:identifier](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#identifier) | Identifier |
| `imdb_id` | External IMDb identifier | [dc:identifier](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#identifier) | Identifier |
| `classification` | Corpus-specific structural format of the object (audiovisual resource) | [dcterms:type](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/type) | Genre |
| `country` | Country/countries of origin; ISO 3166-1 alpha-2 | [dcterms:spatial](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#spatial) | Country of Reference |
| `year` | Year of initial release in the country of origin | [dcterms:issued](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#issued) | Year of Reference |
| `director` | First and last name of the director(s); multiple directors separated by semicolon | [dcterms:creator](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/creator) | Credits |
| `runtime_min` | Runtime in minutes, human-readable | [dcterms:extent](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#extent) | Original Duration |
| `duration_iso8601` | Runtime following ISO 8601 duration format, machine-readable | [dcterms:extent](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#extent) | Original Duration |
| `season_episode` | Season and episode number for serial formats | [dcterms:isPartOf](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#isPartOf) | Series / Serial |
| `episode_title` | Original title of the series episode in its original language | [dc:title](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#title) | Title |
| `modes_intervention` | Project-specific analysis categories | –– | –– |
| `annotation_data` | Related annotation data | [dcterms:relation](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#relation) | Relationship |
| `moviebarcode` | Related moviebarcode | [dcterms:relation](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#relation) | Relationship |

> [!NOTE]
> **Mapping note:** Not all metadata elements in this dataset can be mapped directly to external standards.  
> Some elements intentionally combine multiple conceptual dimensions in order to reflect the internal structure of the corpus. As a result, they cannot always be aligned with a single standardised field.

## Example

An example metadata record for an audiovisual resource is structured as follows:

 {
    "title": "Everything Will Change",
    "object_id": "f029e",
    "imdb_id": "tt13086274",
    "classification": "documentary",
    "country": "DE; NL",
    "year": 2021,
    "director": "Marten Persiel",
    "runtime_min": "93 Min.",
    "duration_iso8601": "PT1H33M",
    "season_episode": "",
    "episode_title": "",
    "modes_intervention": "Post-anthropocentric reperspectivations",
    "annotation_data": true,
    "moviebarcode": true
  }

## Modes of Intervention 

The modes of intervention are defined across seven scalable categories that identify reoccuring audiovisual patterns within the corpus material to examine and analyse strategies through which films and videos modulate forms of world-making in the context of ecological crisis.

1. **Destabilization / Restabilization**
How do media productions relate to the systemic changes of the atmosphere, lithosphere, hydrosphere, and biosphere, and do they make these changes tangible as destabilizations of concrete environments in their spatiotemporal coordinates, in processes of movement and action? Are the possibilities of adaptation or resilience staged as audiovisual dynamics? Polar regions and glaciers for example are popular topoi for framing the destabilization of concrete environments as destabilizations of the coordinates of our perception. 

2. **Escalation / de-escalation**
In climate research and climate communication, there is frequent talk of tipping points and self-reinforcing processes that need to be prevented or mitigated. In which temporal structures of audiovisual images, in which metaphors, are such dynamics made narratively and expressively effective? The tipping points and feedback loops of the climate find their image again and again in the logic of escalation of the montage sequences of activist documentaries. Their counterparts are the expressive and narrative building blocks of the climate disaster movie. However, this experience of escalation and reversal is not seldom countered by dramaturgies of regaining agency, of acceptance or complacency.

3. **Disruptive scaling**
How are the interferences of local and global processes, micro- and macrostructures, individualities and systems, processuality and latency, eventfulness and longue durée staged in audiovisual worlds? A term that has been brought into play more and more recently in the context of the Anthropocene discourse is “scale”. Do films use disorienting scaling effects, where large and small, above and below, man-made and nature-made repeatedly become indistinguishable? Is scaling-up as a mode of the sublime problematic as it produces the sovereign distance of the reflecting subject?

4. **Non-linear temporalities**
The climate crisis is characterized by a complex interaction of after-effects of long past,
current and possible future activities and processes as well as through unforeseeable feedback phenomena. How is it shown in the films that linear causalities and systemic circularity of ecology interfere with each other? How are mutually intervening pasts and futures made audiovisually conceivable? 

5. **Post-anthropocentric reperspectivations**
How do the media products deal with the challenge of discarding anthropocentric worlds and designing bio- or ecocentric worlds? Is there an audiovisual projection of a different kind of collective subject that captures participatory responsibility for interspecies interconnectedness instead of an individualistic, ethical immediacy? Can the films, videos, and series project worlds as manifolds of perspectives and modes of existence?

6. **Unintended Consequences**
In addition to the temporal and spatial scaling, the distributed, diffuse causation and thus responsibility is a problem in making climate change visible and audible. Because no single activity can be determined as causally responsible, individual actions cannot be fixed linearly to the actions of the human species or of Western societies as a whole. How do films deal with this becoming uncanny of agency?

7. **Collisions**
Many media productions, especially nature documentaries, still conceptualise nature as separate from the human realm. This dualism between nature and culture as a root cause of the climate crisis is increasingly manifesting as audiovisual boundaries: wild rainforest bordering cultivated palm tree plantations, fishing nets turning habitat into commodity, dams cutting off waterways for animals to redistribute water for agriculture. Could the clash between geometrically constructed human infrastructure and the chaotic force of extreme weather events make implicit structures of world projections more tangible and thereby challenge these extractivist approaches to nature? How can the interconnectedness of all life-forms be envisioned beyond the Western binary?

## Adiitional Metadata-Files

Along with the corpus metadata there are two additional metadata files:

1. `annotation metadata` for the annotation datasets
2. `moviebarcode metadata` for the moviebarcodes

The primary linking element is the project-specific identifier, the `object_id`. Each `azp` file and each `png` file is assigned a unique identifier derived from the `object_id`, allowing it to be linked to the corresponding audiovisual resource. The metadata also include additional information, such as the date of creation and technical specifications.