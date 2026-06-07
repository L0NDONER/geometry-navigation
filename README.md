# geometry-navigation

Outdoor postcode routing and indoor floor-mesh pathfinding engine for courier and complex-building navigation.

## What it does

Two routing modes in one system:

**Outdoor** — optimises a multi-stop courier manifest using bubble-based sequencing with throat awareness. Understands road patterns (cul-de-sacs, through-roads, farm spurs), delivery side, and entry/exit discipline. Geocodes addresses against a postcode database and projects to local metres for fast spatial ops.

**Indoor** — navigates a building's floor mesh via BFS over a node/edge graph. Handles multi-floor routes through lifts and stairs, cluster entries, closed bays, and height-restricted tunnels. Complexes are defined in `complexes/*.json` using a campus schema (throats, spine, clusters, farm spurs, floors, vertical links).

## Modules

| File | Role |
|---|---|
| `route_optimiser.py` | Bubble clustering, throat classification, A-B-C street sequencing |
| `courier_gps.py` | Van360 geometry, GPS tick processing, dwell detection |
| `geocoder.py` | Address → Vec2 projection against postcode centroid |

## Data schemas

### Postcode (`postcodes/*.json`)
One file per postcode unit. Delivery metadata: pattern, entry/exit preference, landmarks, throat type, internal order, GPS breadcrumbs.

### Area (`areas/*.json`)
Groups of postcodes with visit history, preferred entry/exit, observed neighbours.

### Complex (`complexes/*.json`)
Campus-level schema for large sites (hospitals, factories). Top-level keys:

- `repo` — source repository URL
- `campus` — throats, preferred throat, spine, clusters, farm spurs, terminals
- `floors` — per-floor node/edge graphs with z-levels (`LB1` sub-basement through `L2` second floor)
- `vertical_links` — lift and stair cores connecting floors

**Cluster fields:** `name`, `aliases`, `door_id`, `side`, `floor`, `priority`, `constraints` (e.g. `max_height_m`), `note`

**Floor node types:** `cluster_entry`, `junction`, `room`, `ward`, `lift`, `stair`

**Vertical link types:** `lift`, `stairs` — each core lists its stops in z-order

#### Example: NNUH (`complexes/NNUH.json`)

| Floor | z | Cluster entries |
|---|---|---|
| LB1 Sub-Basement | -1 | Radiology_Main (MRI, CT) |
| L0 Ground | 0 | ED_Main · Bay_04 · Loading_Bay_A · Loading_Bay_B · Service_Tunnel_A |
| L1 First | +1 | — (Ward_1A, Ward_1B, Outpatients) |
| L2 Second | +2 | MHU_Entry (Mental Health Unit) |

Vertical cores: Lift A (LB1→L2), Lift B (L0→L2), Stair B (LB1→L2)

## Pathfinding

BFS over the floor mesh. Nodes: rooms, junctions, lift landings, cluster entries. Edges: walkable connections per floor. Vertical movement via `vertical_links` (lift/stairs). Blocked nodes (closed bays, maintenance) are excluded from traversal at query time — the engine automatically falls back to the next viable entry point.

Entry fallback priority: primary dock → secondary dock → side entrance → service tunnel → last resort (A&E, with restrictions noted in schema).

## Usage

```python
from route_optimiser import Stop, optimise_route
from courier_gps import Vec2
from geocoder import geocode_address
```

The engine is transport-layer agnostic — wire it to any API, CLI, or service. The web layer lives separately in the host application.

## Licence

Copyright (C) 2026 L0NDONER

This program is free software: you can redistribute it and/or modify it under the terms of the GNU Affero General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the [GNU Affero General Public License](LICENSE) for more details.

**Network use clause:** If you run a modified version of this engine as a networked service, you must make the complete source of your modified version available under the AGPL v3.

## Inception

`2026-06-07T10:41:45Z`
