# Release Notes

## Changes in Splinter 0.3.7

### Highlights:
* The admin service and the scabbard service can now send catch-up events to
  bring new subscribers up to date

### Deprecations and Breaking Changes:
* The splinterd --generate-certs flag, which was deprecated in 0.3.6, is still
  available by default. In 0.3.8, the flag will not be available by default.
  Instead, you must use the Rust compile-time feature “generate-certs” to
  explicitly enable the deprecated --generate-certs flag. For more information,
  see the 0.3.6 release notes.
* In the next release, the splinter CLI name will change from “splinter-cli” to
  “splinter”. “splinter-cli” will exist as an alias for “splinter”, but should
  be considered deprecated as of release 0.3.8.

### libsplinter:
* Refactor the admin service and scabbard service to separate the REST API code
* Change admin service events to include a timestamp of when the event occurred
* Update the admin service to send all historical events that have occurred
  since a given timestamp when an app auth handler subscribes
* Update the scabbard event format to correlate directly with a transaction
  receipt
* Remove EventHistory from the REST API because it is no longer used
* Remove EventDealer from the REST API and replace it with the EventSender
* Update the event sender to send catch-up events as an asynchronous stream
* Add state-delta catch-up to the scabbard service, sending all events that
  occurred since a given event ID when a subscription request is received
* Update Network to properly clean up connections on disconnection

### splinterd:
* Fix the splinterd --heartbeat argument to properly accept a value

### Gameroom example:
* Update the gameroomd app auth handler to track the timestamp of the last-seen
  admin event
* Add the timestamp for the last-seen admin event to the app auth handler’s
  subscription request for getting any catch-up events
* Update the gameroom state-delta subscriber to track the ID of the last-seen
  scabbard event
* Add the ID of the last-seen scabbard event when subscribing to scabbard on
  restart, which lets the gameroom daemon receive catch-up events

### Private XO example:
* Replace the transact git repo dependency with a crates.io dependency

## Changes in Splinter 0.3.6

### Highlights:
* Peers can now successfully reconnect after restarting
* Faster build times:
  * Added a Dockerfile for splinter-dev docker image
  * Based the installed images on the splinter-dev image
  * Updated the compose files to build the installed docker images
  * Added parallelization to Travis CI builds
* Initial database structure for Biome, the libsplinter module that supports
  user management
* New Gameroom Technical Walkthrough document that explains the Splinter
  functionality that powers the Gameroom application; see
  examples/gameroom/README.md for a link to the PDF

### Deprecated Features:
* The --generate-certs flag for splinterd is now deprecated. Instead, please
  generate development certificates and keys using the new command "splinter-cli
  cert generate". This command will generate the certificates and keys in
  /etc/splinter/certs/ (by default) or in the specified directory.
  Note:  --generate-certs is still available by default in 0.3.6. It will be
  turned off in 0.3.7, but will still be available with a Rust compile-time
  feature flag. If using generated certificates, run splinterd with the
  --insecure flag.

### libsplinter:
* Improve logging:
  * Log when a peer is removed
  * Log an event Reactor background thread error on startup
  * Log REST API background thread startup errors immediately, rather than on
    shutdown
  * Log a WebSocket shutdown
  * Log a peer connection initiation
  * Log the configuration used to start splinterd
  * Add timestamp and thread name to log messages
* Return an error when a peer is disconnected
* Allow consensus threads to log error and exit, rather than panic
* Enforce that member, endpoint, and service IDs are unique to a circuit
* Update the example TOML configuration file
* Verify a CircuitManagementPayload message's payload field, header field, and
  payload signature
* Update example circuit files to use correct enum types
* Fix a typo in DurabilityType enum
* Stop the admin service once a shutdown signal is received
* Fix a locking bug that prevented admin service from properly shutting down
* Stop running services upon admin service shutdown
* Include the service definition in service shutdown error
* Update format lint for Rust 1.39
* Add a Splinter PostgreSQL database to be used by Splinter modules
* Decouple EventDealer and EventHistory to allow the storage of events to be
  managed separately from event history
* Change the log levels of received messages and pings/pongs
* Update EventDealers to return error from EventDealer.add_sender method and
  handle errors from EventDealer.dispatch method
* Store AuthServiceEvents in a Mailbox, replacing LocalEventHistory
* Start reorganizing the admin service module
* Store pending changes as transaction receipts in scabbard
* Add a "state_" prefix to variables that refer to the scabbard LMDB backend
  database, which helps distinguish this database from other databases that
  scabbard may maintain
* Run tests behind the "experimental" feature
* Move the zmq-transport feature, which loads the ZMQ dependency, to experimental
* Rename the node registry method create_node add_node, which more accurately
  reflects its functionality
* Update the struct used to build REST API resources to represent multiple
  method and handler pairs for a given resource
* Fix the node registry implementation’s file editing to completely overwrite
  the YAML node registry file rather than append changes
* Add a disconnect listener to Network; this listener is used to close the
  connection when a peer is disconnected from the network
* Register the AuthorizationManager to listen for peer disconnections to clean
  up old state about the disconnected peer

### splinterd:
* Add endpoints for local registry, including:
  * POST /nodes
  * DELETE /nodes/{identity}
  * PATCH /nodes/{identity}
* Move the node registry implementation from splinterd to libsplinter
* Update the struct used to build REST API resources to represent multiple
  method and handler pairs for a given resource
* Run tests behind the experimental feature
* Add /circuits route, available with circuit-read experimental feature,
* Update splinterd to look for certificates and keys in /etc/splinter/certs (by
  default) or the location specified by "--cert-dir" or the environment variable
  SPLINTER_CERT_DIR
* Deprecate the generate-cert flag (will be removed in a future release) now
  that "splinter-cli cert generate" is available

### splinter-cli:
* Add subcommand "cert generate" to generate certificates and keys that can be
  used to run splinterd for development.

### Canopy:
* Add CSS styles for responsive side navigation bar
* Add default color styles to be used in design app
* Add default typography styles and initial typography documentation
* Add CSS class defaults and themes for navigation
* Add structure and initial introduction page for the documentation app
* Add configuration to build theme CSS bundles
* Add the initial structure for a sapling example (an application to extend
  Canopy)
* Implement register and initialize functions for saplings in CanopyJS
* Add lint and unit tests to Travis CI
* Refactor CanopyJS to improve clarity and extensibility
* Implement CanopyJS user

### Gameroom example:
* Add a generic-themed Gameroom app to installed docker-compose file
* Add functions to check for active gamerooms and resubscribe on startup
* Add volumes for /var/lib/splinter to the docker-compose file
* Add timestamp and thread name to log messages
* Remove the hardcoded protocol for octet-stream submission; instead, use a
  relative URL handled by the proxy
* Attempt to reconnect WebSocket clients if a "close" message is received
* Time out WebSocket client connections and attempt to reconnect
* Convert signature hex string to bytes for signing payloads
* Base the test docker image on the splinter-dev docker image
* Fix a bug with cell selection

### Packaging:
* Remove known errors during a .deb package install

## Changes in Splinter 0.3.5

### Highlights:
* Add network-level heartbeats to improve peer connectivity
* Update Gameroom UI to use the WebSocket Secure protocol (wss) when the
  application protocol is HTTPS
* Improve libsplinter tests
* Add code of conduct to README
* Add the command-line option --common-name to splinterd

### Canopy:
* Add initial directory structure for the Canopy project, a web application
  that hosts pluggable applications and tools built on Splinter

### Gameroom example:
* Remove unnecessary logo files
* Update UI to use wss when the application protocol is HTTPS. This fixes an
  issue where the application could not communicate via WebSockets if the
  application was communicating over HTTPS
* Check for batch status after batch is submitted, then wait for batch to be
  committed or invalidated in gameroomd
* Remove member node’s metadata from gameroom propose request payload
* Fetch member node information from splinterd when gameroomd receives a
  gameroom propose request

### libsplinter:
* Add dockerfile for libsplinter crate generation
* Document the limitations for two-phase commit
* Add network-level heartbeats. The network now creates a thread that will send
  a one-way heartbeat to each connected peer every 30 seconds by default.
* Rename libsplinter crate to splinter
* Store the current state root hash for scabbard's shared transaction state in
  order to support restarts
* Simplify where services can be connected. This ensures that a service is
  connected to the first allowed node and that allowed nodes can only have one
  service.
* Remove peers when a node is disconnected


### libsplinter Testing:
* Update key_not_registered test to use a valid circuit
* Rename error_msg to msg in AdminDirectMessage tests
* Correctly set message type to CircuitMessageType::CIRCUIT_DIRECT_MESSAGE in
  AdminDirectMessage tests
* Fix typos in doc comments

## Changes in Splinter 0.3.4

### Highlights
* Implement a batch status endpoint for scabbard
* Set up the Cypress integration test framework for the Gameroom UI

### Gameroom example
* Copy Splinter .proto files into installed client builds
* Redirect the user to a “Not Found” page if the page does not exist
* Set up integration tests using Cypress
* Add XO smart contract to installed gameroomd builds

### libsplinter
* Reduce latency of events by replacing run_interval in EventDealerWebSocket
  with streams

### scabbard
* Change the scabbard database name to be the sha256 hash of
  service_id::circuit_id to ensure that it will be a valid file name
* Add signature and structure verification to the scabbard service when it
  receives batches submitted via the REST API
* Add /batch_statuses endpoint to scabbard and update /batches endpoint to
  return a /batch_statuses link for the submitted batch IDs

### splinterd
* Add config builder with toml loading (experimental feature)

### Packaging
* Add Dockerfile to package gameroom UI
* Update packaging for gameroomd and splinterd so modified systemd files are
  not overwritten
* Modify gameroomd and splinterd postinst scripts to add data directories
* Add plumbing to properly version deb packages

## Changes in Splinter 0.3.3

### Highlights
* Add functionality to create and play XO games

### libsplinter
* Add EventHistory trait to EventDealer to allow for new event subscribers to catch
  up on previous events. This trait describes how events are stored.
* Add LocalEventHistory, a basic implementation of EventHistory that stores events
  locally in a queue.
* Add MessageWrapper to be consumed by EventDealerWebsockets, to allow for
  shutdown messages to be sent by the EventDealer
* Enforce that a Splinter service may only be added to Splinter state if the
  connecting node is in its list of allowed nodes
* Add Context object for WebSocket callbacks to assist in restarting WebSocket
  connections
* Add specified supported service types to the service orchestrator to determine
  which service types are locally supported versus externally supported
* Only allow initialization of the orchestrator’s supported service
* On restart, reuse the services of circuits which are stored locally
* Add circuit ID when creating a service factory, in case it is needed by the
  service
* Replace UUID with service_id::circuit_id, which is guaranteed to be unique on
  a Splinter node, to name the Scabbard database
* Fix clippy error in events reactor
* Fix tests to match updated cargo args format
* Change certain circuit fields from strings to enums
* Remove Splinter client from CLI to decrease build time

### Gameroom Example
* Add ability in the UI to fetch and list XO games
* Correct arguments used to fetch the members of an existing gameroom, allowing
  the members to be included in the /gamerooms endpoint response
* Add GET /keys/{public_key} endpoint to gameroomd, to fetch key information
  associated with a public key
* Add UI functionality to create a new XO game:
* Add ability to calculate addresses
* Add methods to build and sign XO transactions and batches
* Add methods to submit XO transactions and batches
* Add form for user to create new game
* Add new game notification to UI and gameroomd
* Add player information displayed for a game in UI
* Implement XO game board in UI
* Implement XO take functionality and state styling in UI
* Add component to show game information in the Gameroom details page in the UI
* Use md5 hash of game name when creating a game, rather than URL-encoded name
  that handles special characters
* Add player information when updating an XO game from exported data (from state
  delta export)
* Add auto-generated protos for the UI
* Remove the explicit caching in the Gameroom Detail view in the UI, because Vue
  does this automatically
* Make various UI styling fixes
* Remove unused imports to avoid cargo compilation warnings

## Changes in Splinter 0.3.2

### Highlights
* Completed the code to propose, accept, and create a gameroom in the Gameroom
  example application

### libsplinter
* Persist AdminService state that includes the pending circuits
* Replace the WebSocketClient with a new events module, which improves
  multi-threaded capabilities of the clients (libsplinter::events; requires the
  use of "events" feature flag)
* Improve log messages by logging the length of the bytes instead of the bytes
  themselves
* Fix issue with sending and receiving large messages (greater than 64k)
* Fix issues with threads exiting without reporting the error
* Removed inaccurate warn log message that said signature verification was not
  turned off

### splinterd
* Add Key Registry REST API resources
* Increase message queue sizes for the admin service's ServiceProcessor.

### splinter-cli
* Remove outdated CLI commands

### Gameroom Example
* Add XoStateDeltaProcessor to Gameroom application authorization handler
* Add route to gameroom REST API to submit batches to scabbard service
* Set six-second timeout for toast notifications in the UI
* Add notification in the UI for newly active gamerooms
* Enhance invitation UI and add tabs for viewing sent, received, or all
  invitations
* Fix bug that caused read notifications to not appear as read in the UI
* Fix bug where the Gameroom WebSocket was sending notifications to the UI
  every 3 seconds instead of when a new notification was added

## Changes in Splinter 0.3.1

### Highlights

* Completion of circuit proposal validation, voting, and dynamic circuit creation
* Addition of key generation and management, as well as role-based permissions
* Continued progress towards proposing, accepting, and creating a gameroom in the
  Gameroom example application

### libsplinter

* Add AdminService, with support for:
  * Accepting and verifying votes on circuit proposals
  * Committing approved circuit proposals to SplinterState
* Add notification to be sent to application authorization handlers when a
  circuit is ready
* Update scabbard to properly set up Sabre state by adding admin keys
* Add support for exposing service endpoints using the orchestrator and service
  factories
* Add WebSocketClient for consuming Splinter service events
* Add KeyRegistry trait for managing key information with a StorageKeyRegistry
  implementation, backed by the storage module
* Add KeyPermissionsManager trait for accessing simple, role-based permissions
  using public keys and an insecure AllowAllKeyPermissionManager implementation
* Add SHA512 hash implementation of signing traits, for test cases
* Add Sawtooth-compatible signing trait implementations behind the
  "sawtooth-signing-compat" feature flag.

### splinterd

* Add package metadata and license field to Cargo.toml file
* Add example configuration files, systemd files, and postinst script to Debian
  package
* Reorder internal service startup to ensure that the admin service and
  orchestrator can appropriately connect and start up
* Use SawtoothSecp256k1SignatureVerifier for admin service

### splinter-cli

* Add "splinter-cli admin keygen" command to generate secp256k1 public/private
  key pairs
* Add "splinter-cli admin keyregistry" command to generate a key registry and
  key pairs based on a YAML specification

### Private XO and Private Counter Examples
* Add license field to all Cargo.toml files
* Rename private-xo package to private-xo-service-<version>.deb
* Rename private-counter packages to private-counter-cli-<version>.deb and
  private-counter-service-<version>.deb

### Gameroom Example
* Add package metadata and license field to gameroomd Cargo.toml file
* Add example configs, systemd files, and postinst script to gameroomd Debian
  package; rename package to gameroom-<version>.deb
* Implement notification retrieval using WebSocket subscription and
  notifications endpoints
* Show pending and accepted gamerooms in the Gameroom UI
* Add full support for signing CircuitManagementPayloads with the user's
  private key and submitting it to splinterd
* Update gameroomd to specify itself as the scabbard admin and submit the XO
  smart contract when the circuit is ready
* Make various UI enhancements

## Changes in Splinter 0.3.0

### Highlights

* Completion of the two-phase-commit consensus algorithm with deterministic
  coordination
* Continued progress towards dynamically generating circuits, including
  dynamic peering and circuit proposal validation
* Continued progress on the Gameroom example, including UI updates and
  automatic reconnection

### libsplinter

* Add a service orchestration implementation
* Add Scabbard service factory 
* Implement a deterministic two-phase-commit coordinator
* Reorder the commit/reject process for the two-phase-commit coordinator. The
  coordinator now tells proposal manager to commit/reject before broadcasting
  the corresponding message to other verifiers.
* Refactor two-phase-commit complete_coordination. Move the process of 
  finishing the coordination of a proposal in two-phase commit to a single
  function to reduce duplication.
* Implement a two-phase-commit timeout for consensus proposals
* Update the two-phase-commit algorithm to ignore duplicate proposals
* Allow dynamic verifiers for a single instance of two-phase-commit consensus
* Add an Authorization Inquisitor trait for inspecting peer authorization state
* Add the ability to queue messages from unauthorized peers and unpeered nodes
  to the admin service
* Fix an issue that caused the admin service to deadlock when handling proposals
* Add Event Dealers for services to construct websocket endpoints
* Add a subscribe endpoint to Scabbard
* Validate circuit proposals against existing Splinter state
* Update create-circuit notification messages to include durability field

### splinterd

* Log only warning-level messages from Tokio and Hyper
* Improve Splinter component build times
* Add a NoOp registry to handle when a node registry backend is not specified

### Private XO and Private Counter Examples

* Use service IDs as peer node IDs, in order to make them compatible with
  two-phase consensus

### Gameroom Example

* Add server-side WebSocket notifications to the UI 
* Add borders to the Acme UI
* Improve error handling and add reconnects to the Application Authorization
  Handler
* Add a circuit ID and hash to GET /proposals endpoint
* Standardize buttons and forms in the UI
* Improve error formatting in the UI by adding toasts and progress bar spinners
* Change the Gameroom REST API to retrieve node data automatically on startup
* Split the circuit_proposals table into gameroom and gameroom_proposals tables
* Use the [Material elevation strategy](https://material.io/design/color/dark-theme.html)
  for coloring the UI
* Decrease the font size
* Change the UI to redirect users who are not logged in to login page
* Add a dashboard view
* Add an invitation cards view
* Add a button for creating a new gameroom to the UI

## Changes in Splinter 0.2.0

### libsplinter

* Add new consensus API (libsplinter::consensus)
* Add new consensus implementation for N-party, two-phase commit
  (libsplinter::consensus::two_phase)
* Add new service SDK with in-process service implementations
  (libsplinter::service)
* Add initial implementation for Scabbard, a Splinter service for running Sabre
  transactions with two-phase commit between services
(libsplinter::service::scabbard)
* Add REST API SDK (consider this experimental, as the backing implementation
  may change)
* Add new node registry REST API endpoint for providing information about all
  possible nodes in the network, with initial YAML-file backed implementation.
* Add new signing API for verifying and signing messages, with optional
  Ursa-backed implementation (libsplinter::signing, requires the use of
"ursa-compat" feature flag)
* Add MultiTransport for managing multiple transport types and selecting
  connections based on a URI (libsplinter::transport::multi)
* Add ZMQ transport implementation (libsplinter::transport::zmq, requires the
  use of the "zmq-transport" feature flag)
* Add peer authorization callbacks, in order to notify other system entities
  that a peer is fully ready to receive messages


### splinterd

* Add REST API instance to provide node registry API endpoints
* Add CLI parameter --bind for the REST API port
* Add CLI parameters for configuring node registryy; the default registry type
  is "FILE"

### Gameroom Example

* Add gameroom example infrastructure, such as the gameroomd binary, docker
  images, and compose files
* Add Login and Register UI
* Add New Gameroom UI
* Add UI themes for both parties in demo
* Initialize Gameroom database
* Add circuit proposals table
* Initialize Gameroom REST API
* Implement Gameroom REST API authentication routes
* Implement Gameroom REST API create gameroom endpoint
* Implement Gameroom REST API proposals route
* Implement /nodes endpoint in gameroomd
