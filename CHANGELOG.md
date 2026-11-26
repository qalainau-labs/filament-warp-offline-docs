# Changelog

All notable changes to `qalainau/filament-warp-offline` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.2.2] - 2026-11-26

### Fixed

- Related lists taken offline (relation managers) now break ties on the table's default sort column by primary key, in the same direction. Before, rows with the same sort value could come back in a different order on PostgreSQL, so which rows made it under the row limit could change from one sync to the next.
- Relationship select options taken offline (up to 1,000) are now read in primary key order, so the same options are kept on every sync when there are more than 1,000.

## [1.2.1] - 2026-11-25

### Fixed

- **Text typed in a `ScanInput` before its script finished loading was lost**: the field loads its script when it appears, and on a slow connection or a busy device the first characters typed (or a click on “Create” right away) went nowhere: the field was emptied when the script arrived, and the record could be saved without them. Those characters now reach the form and stay in the field. A re-render while the script was still loading (for example a validation error) also no longer shows the JavaScript error `state is not defined`. Nothing to change in your app.

## [1.2.0] - 2026-11-25

### Added

- `window.WarpOffline.whenIdle()`: a promise that resolves when the device has finished sending and reading (including a sync that starts while it waits). Browser tests can wait for a sync with it instead of a fixed pause. See *Testing it* in the README.

### Fixed

- **A confirmed page button could do nothing after the connection dropped during a sync.** When the reply to a sync request that was already on its way arrived after the connection dropped, the status bar switched back to *Online*. A page button with a confirmation (for example *Clock out*) was then sent to the server instead of the device: no confirmation box appeared and nothing happened. A reply now only marks the device online when the connection did not drop while it was on its way and the browser reports a connection.

## [1.1.0] - 2026-11-22

### Fixed
- **A device's first report could fail with HTTP 500**: with the device list on (`devices()`), when the first two reports of a new device arrived at the same time (for example right after preparing it), both tried to add the device and the second one failed on the unique device ID. The second report now updates the row the first one added.

- **The panel's root page (for example `/admin`, a page with the slug `/` and `HasOfflineActions`) could not be opened offline.** The service worker's scope ended with a slash (`/admin/`), so `/admin` itself was outside it: the page was never saved and showed the browser's offline error. The scope is now `/admin` (the manifest scope and `Service-Worker-Allowed` too), and the home screen still opens `/admin/`. A device registers the new scope the next time the panel is opened online; the root page is saved from the visit after that. Because a scope is a text prefix, a page whose address only begins with the panel's (`/admin-reports`, `/administrator`) is also under the service worker; Warp Offline leaves those pages and their files alone.
- On a page with offline buttons (`HasOfflineActions`), the offline status bar was shown again after every button press on a server that reuses one application for many requests (for example Pest browser tests), so a time clock page could show two, three or more bars. The page now always shows one.
- The confirmation box of a page button (`OfflinePageAction`) used the plugin's own button text while offline (Japanese: *実行する*) and Filament's text while online (Japanese: *確定*). Both boxes now use the same text: Filament's *Confirm* and *Cancel*, or your own `modalSubmitActionLabel()` / `modalCancelActionLabel()`.
- **Page access rules on a replaced page**: when the app replaced a standard page or resource with a subclass (`xxxPage()`/`xxxResource()`), `canAccessUsing()` and `canWriteUsing()` received the subclass as `$page`, so a rule such as `$page !== Projects::class` did not match and a page the app had closed still opened (HTTP 200). `$page` is now always the plugin's own standard class, whatever the app swaps in. A rule that compared `$page` with the app's own subclass must compare with the standard class instead.

### Added

- `offlinePage()` replaces the *Offline* page with your own class (a subclass of `OfflinePage`), keeping the same address and route name. `getOfflinePage()` returns it. `offlinePage(false)` takes the page off, like `page(false)`. See *Your own Offline page* in the README.

### Documentation

- The README's *Limits* now say that a photo saved on the device cannot be edited with Warp Painter without a connection: Painter's editing screen and its save need the server.

## [1.0.2] - 2026-11-20

### Fixed

- The confirmation box of a page button (`OfflinePageAction`, for example *Clock out* on a time clock page) was see-through while offline: the page text behind it showed through the box and overlapped its text, in light and dark mode. The box now has its own white (dark: gray) background wherever it is shown.

## [1.0.1] - 2026-11-20

### Fixed

- Changing the language while *Work offline* is on (for example with a language menu that reloads the page) left the list's column headings, badges, dates and the create button on the device in the previous language. The device now reads the list again in the new language when the page opens, while still working offline. A really offline phone still shows the previous language until its next sync.
- French: the *En attente d'envoi* label of the offline list used a straight apostrophe, unlike the rest of the translation.

## [1.0.0] - 2026-11-20

### Added

- **Buttons on any page, offline.** `OfflinePageAction`, `HasOfflineActions`, `OfflinePress` and `OfflinePendingPresses` queue page buttons with their pressed time, keep local button state, sync once with authorization, and show offline marks and receipt times in the change log. The pages also show the offline status bar. Try it on the demo’s Time clock page.
- The change log and devices lists also ask your app's policy (`viewAny`) when you have one for the model, and removing a device asks `delete`.
- **Restricting the Offline page per role.** `canAccessUsing(fn (User $user, string $page): bool => …)` and `canWriteUsing(…)` on the plugin let your app decide who can open the Offline page (menu, URL and Livewire calls) and who only looks at it. Without them nothing changes.
- **A checklist whose points depend on the record.** `Checklist::itemsFor('service_type_id', fn (): array => [1 => [...], 2 => [...]])` picks the points by another field of the form. The device gets every set and picks one by the record's value, and changes the points at once when that field changes in the form (the online form too). The server checks and saves with the points of the value being saved.
- **Offline actions in the online table.** `HasOfflineSupport::offlineActions()` (or `OfflineAction::toAction()`) makes Filament row actions with the same label, confirmation, `visibleWhen()` and `set()`, applied like a sync (permissions, the form's rules, `mutateBeforeSave()`, `using()`, `afterSync()`). `OfflineAction::run($resource, $record)` applies one in your own code. `OfflineAction::successNotificationTitle()` sets the notification of the online button.
- **Read-only rows.** A row whose record the person cannot edit (the resource's `canEdit($record)`) shows *View* instead of *Edit* in the offline list, opens read-only, and has no offline actions or editable cells.
- **Drawing on a photo, offline.** With Warp Markup 1.4 or later installed, its `MarkupField` works in the offline form: arrows, circles, pen strokes and numbered pins with notes on the first photo of a `FileUpload` (`->imageFrom('photos')`, also a photo that was just taken and has not been sent), or on a fixed picture (`->image('…')`, stored on the device by *Prepare to go offline*). The marks wait on the device with the record and are checked on the server like in the online form. Fields whose picture is decided for each record by a closure stay read-only offline.
- **Cards and swipes on phones, with Warp Mobile.** When Warp Mobile is installed and the resource's table has `mobileCards()`, the offline list shows the same cards on narrow screens (title, subtitle, badge, picture, *Label: value* lines, hidden columns) and `swipeActions()` works offline: offline actions, `edit`, `view` and `delete` become swipe buttons, a full swipe runs the first one (with its confirmation), and names that only work online are left out. Without Warp Mobile, or without `mobileCards()`, the offline list does not change.
- Device list for administrators (`->devices(canView: …)`, off by default): every device that holds offline data, with its user, the changes waiting on it, the changes to fix and the last sync. *Delete data on the device* empties a lost device the next time it reaches the server — also when it is signed out — signs it out and tells the person. Adds the `warp_offline_devices` table to the migration.
- Related lists offline, read-only: `relationManagers()` in `offline()` carries the rows of the resource's relation managers with each record (the columns and sort of their `table()`, their own query, `canViewForRecord()`), and shows them under the offline form — open a piece of equipment and read its past inspections. A resource with a `view` page gets **View** next to **Edit** in the offline table: the record opens read-only, also from its view URL without a signal.
- `Checklist`: a form field for inspection checklists — *OK*, *Not OK* or *N/A* for each point, with a note and photos per point, saved in one JSON column. Works in the online form (photos through Livewire) and offline (photos wait on the device). `required()` asks for every point, `noteRequiredOnFail()` for a note on failed points; `Checklist::count()` adds up the results.
- `Repeater` works offline: rows are added, removed and reordered on the device, with the fields, defaults and rules of the row schema, for a JSON column and for a `relationship()` on a has-many (with `orderColumn()`). The server checks every row and the number of rows, updates, creates and deletes only rows of the record's own relationship, and refuses added or removed rows when the repeater is not `addable()` / `deletable()`. Before, a repeater was shown as text offline.
- Fields that depend on other fields keep working offline: `visible()`, `hidden()`, `required()`, `disabled()` and `options()` closures that read other fields with `Get` — on a field or on the section around it — are tried with every value those fields can have, and the device looks the answer up as the form changes (“ask what is wrong only when the result is *Failed*”). `visibleJs()` and `hiddenJs()` are evaluated on the device. On the server, the real closures are evaluated with the values sent: values for hidden fields are dropped and fields required for those values must be filled. Before, such conditions were evaluated once, for an empty form.
- Offline list pages: register a resource and its own `table()` and `form()` keep working without a signal, drawn in the browser from the records on the device. Online, the normal Filament table is used and the device copy is kept up to date in the background.
- Offline table: search, sorting, select and yes/no filters, pages, editable toggle/checkbox/select/text-input columns, bulk selection, rows marked *Waiting to send*, *Not saved* (refused by the server) or *To resolve*.
- Offline form for creating and editing, with sections and grids, common checks on the device, every rule of the field on the server, and drafts kept if the page closes.
- `OfflineAction`: row and bulk actions that set values offline (`@now`, `@today`, `@user`), with conditions, confirmation and server-side code after saving.
- Queue of changes on the device: sent once (each change has an ID, repeats return the first result), in order, edits of new records folded into their creation, several edits of one record folded together.
- Conflicts compared field by field: changes to different fields are merged; for the same field the person chooses per field, or the resource decides (`ConflictStrategy::ServerWins` / `ClientWins`). Records deleted meanwhile can be created again.
- Changes refused by the server (validation, policies, your own code) are listed with the reason and can be fixed in the form and sent again, or discarded.
- Offline scope per resource (`query()`, `limit()`), changes-only sync with regular full refreshes, records that left the scope removed from the device.
- Photos and files for `FileUpload` fields: kept on the device, shrunk before sending, uploaded to the field's disk and directory, once.
- `ScanInput` (QR codes and barcodes with the camera), `Signature` and `Geolocation` form fields, online and offline; a *Scan* button on the offline table.
- Data on the device encrypted (AES-GCM) with a key the browser cannot export; optional or required PIN with automatic locking and deletion after five wrong tries; deleted on logout (asking first when changes are unsent), when another user signs in, and after days without syncing.
- *Offline* page: prepare to go offline (records, pages and photos), data per list, storage, keep data on the device, PIN, delete, install as an app, and the changes on the device.
- Service worker and web app manifest for the panel; pages not set up for offline use show the *Offline* page or a clear notice.
- *Work offline* switch to use the offline table and form while connected.
- Change log (*Offline changes*): who, which device, when it was made on the device and in which time zone, when it arrived, the values and the result.
- `warp-offline:prune` command for unused uploads and old change records.
- Multi-tenancy: endpoints under the tenant, a separate device database per tenant, records attached to the tenant.
- `tenantKey()`: the tenant (company) for panels without Filament's tenancy. The change log, the uploads and the device database use it like a Filament tenant.
- Uploads store their tenant (`tenant_type`, `tenant_id`) and are only used for changes of the same tenant; both tables have an index that starts with the tenant.
- Interface in English, Japanese, German, French, Spanish, Italian, Brazilian Portuguese, Korean and Chinese (simplified and traditional).
