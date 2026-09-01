# FabricNavigator 26.08.10.181

- ACLI settings now use the full available width and grow to their content without an inner vertical scrollbar.
- Topology groups use the same rounded card design as other nodes while retaining their group illustration.
- Port labels avoid both other labels and unrelated link paths.
- Zoom controls and the visible link legend remain available in a sticky toolbar below the topology.
- The unsaved-topology warning can save the current topology before leaving the page.
- Link endpoints are resynchronized after node images are applied so links are centered correctly on initial topology load.
- Persistent FabricNavigator application data now uses `/opt/fabricnavigator/data` instead of the legacy EDM-named directory.
- Existing Docker and appliance installations retain their stored credentials, ACLI components, preferences, discovery settings, routes, and WebView assignments during migration.
- ACLI startup messages now show FabricNavigator paths for SED and alias files.
