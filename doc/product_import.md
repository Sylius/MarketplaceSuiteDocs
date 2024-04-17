## Product importer

Open Marketplace also contains support for importing products via CSV file with elements separated via `|` sign.

Below is an example of supported file structure
```csv
product_code|vendor_id|name|attributes|url|images|description|enabled|taxon_code|auto_verify|prices|shipping_category
test_product|test_uuid|Test product|"[{""type"":""text"",""code"":""TestAttributeCode"",""value"":""TestAttributeValue""}]"|https://example.com|"[{""type"": ""main"", ""url"": ""https://example.com/example.png""}]"|Test product|true|for_home|true|"{""open-marketplace"":{""minimum_price"":123,""original_price"":345,""price"":456},""non-existant-channel"":{""minimum_price"":321,""original_price"":543,""price"":654}}"|default
```

### Usage

Invoke the CLI import with the following command:
```bash
php bin/console open-marketplace:product:import [--batch-size] <filepath>
```

The `<filepath>` is the path to the CSV file used for import, while `--batch-size` (shortcut `-b`, with default value `25`) is used to specify how many products to save into database before performing optimization procedures.

### Customization

To add a new element for the import, create a class extending the `Sylius\MarketplaceSuite\Component\Importer\Product\Handler\AbstractHandler` class.

The class contains two methods, `supports` and `handle`.

The first one verifies if required data is present in the imported file.

Meanwhile, the `handle` method contains the implementation to add imported data to the product entity.

After creating your custom handler, simply register it as a service with tag `open_marketplace.import.product.handler` for it to be included in the import.

### Testing

For testing purpose the best is to run the fixtures, generate an import file and run the importer.

The command, which should help you testing the functionality:

```bash
bin/fixtures && bin/imports-generator --listings-count=1000 && bin/console open-marketplace:product:import --batch-size=100 var/products.csv
```

When running the command above, you may be asked for an import file size (listings count) when not passing the `--listings-count` parameter.

The `bin/imports-generator` executable generates a random import file in `var/` directory. Please use it only for testing purpose. Please do not run it on production. 
