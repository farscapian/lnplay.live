
## BOLT12 Product Offers

Here's how you create the BOLT12 Product Offers:

```bash
./lightning-cli.sh --id=1 -k offer amount=5sat description="8 node environment." quantity_max=1344  issuer="lnplay.live"
./lightning-cli.sh --id=1 -k offer amount=6sat description="16 node environment." quantity_max=2688  issuer="lnplay.live"
./lightning-cli.sh --id=1 -k offer amount=7sat description="32 node environment." quantity_max=5376  issuer="lnplay.live"
./lightning-cli.sh --id=1 -k offer amount=8sat description="64 node environment." quantity_max=10752  issuer="lnplay.live"
```



### Issuing BOLT12 Offers (REQUIRED)

When creating the [BOLT12 Product Offers](https://docs.corelightning.org/reference/lightning-offer), the amount should be set to the price (in sats per node-hour) as specified in the Product Definition.

Note: The `[quantity]` field is an integer representing ONE NODE HOUR (for 8 nodes running for 2 hour, quantity=16). 

Note: Product customizations (OPTIONAL FOR MVP) MAY be passed to the provisioning script using the [payer_note] field `fetchinvoice` (JSON expected).

# Future work

* Implement [lightningaddress for bolt12](https://github.com/rustyrussell/bolt12address) on the backend, allowing the front-end to dynamically fetch the BOLT12 product offers using names (e.g., product-a@domain.tld). This results in the front-end being completely decoupled from the backend and eliminates the need for build-time paramemter in the frontend.