# Xsolla API MCP operations and REST fallbacks

| Step | Product; search query | Original operation IDs |
|---|---|---|
| 1: groups and items | catalog; `create item group` / `virtual item` | `admin-create-group`; `items_create`, `items_get`, `items_update`, `items_delete` |
| 1: currency and packs | catalog; `virtual currency` / `currency package` | `admin-create-currency`, `currency_get`, `admin-update-currency`; `admin-create-currency-package`, `admin-get-currency-package`, `admin-update-currency-package` |
| 1: bundles and games | catalog; `bundle` / `game` | `admin-create-bundles`, `bundles_get`, `admin-update-bundles`; `admin-create-game`, `admin-get-game`, `admin-update-game` |
| 2: pricing | catalog; `update item price` | Relevant create/update above, with `prices` or `vc_prices`; use REST if required fields are unsupported |
| 3: client catalog | catalog; `storefront catalog` / `item groups` / `game key SKU` | `catalog_items_list`, `catalog_currency_list`, `catalog_currency_packages_list`, `catalog_bundles_list`, `catalog_games_list`, `catalog_groups_list`, `catalog_game_key_get` |
| 4: purchases | webshop; `payment order` / `cart` / `virtual currency` / `free` | `payment_create_order_item`; `cart_fill`, `cart_add_item`, `cart_remove_item`, `payment_create_order_cart`, `payment_create_order_cart_id`; `payment_buy_virtual_item`, `free_grant_item`, `free_grant_cart` |
| 4: server token alternative | webshop; `payment token` | `admin-create-payment-token` |
| 5: polling | webshop; `order status` | `order_get` (player Bearer auth); not Payments' Basic-auth operation of the same name |
| 6: verify and clean up | catalog; `delete virtual item` / `delete currency` / `delete bundle` | Reuse typed reads and purchase/tracking above; `items_delete`, `admin-delete-currency-package`, `admin-delete-currency`, `admin-delete-bundles` |

Known gaps — use the documented REST calls in the companion reference files:
- Admin item/bundle lists by group (`admin-list-items-by-group`,
  `admin-list-items-by-group-id`, `admin-list-bundles-by-group`,
  `admin-list-bundles-by-group-id`), game-key upload (`admin-upload-game-codes`),
  regions (`admin-*-region`, `admin-list-regions`), and JSON import
  (`admin-import-catalog`) differ from the public REST routes as of October 2026
  (compare with the [published Catalog OpenAPI](https://developers.xsolla.com/_bundle/api/catalog/index.json?download));
  use REST for them until a newer API MCP release says otherwise.
- Item attributes, pre-order limits and per-user item limit operations also use
  route shapes that differ from the public API as of October 2026; use REST for them.
- Catalog group update/delete are missing. Merchant `groups_update`/`groups_delete`
  use another route and numeric IDs; use Catalog REST with `external_id` instead.
  `admin-grant-entitlement`/`admin-revoke-entitlement` advertise different bodies from
  the public entitlement contract. Game update by numeric ID also requires REST.
- Schemas omit fields such as item limits/periods/regions and top-level purchase
  `sandbox`/`promo_code`. Check support; never omit sandbox mode to fit a schema.
  A 401/403 on an admin list is not an empty catalog: it can be an access problem
  or one of the route differences above, so check access and fall back to REST or
  a known-SKU read. Never retry an uncertain write blindly.
- Only API MCP `items_update` has automatic read/map/merge/write preservation;
  arrays replace arrays, and unsupported or incomplete state blocks the write.
  Other item updates warn about replacement. Map full Admin state to writable fields;
  do not echo a GET response or bypass an adapter refusal with a partial REST PUT.
- MCP calls use the MCP host's IP; public catalog operations do not inject optional
  player JWT personalization. Keep storefront and client purchases in the client.
  Payment UI, webhook handlers, WebSockets, and Event API integration remain app work.
