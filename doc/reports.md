## Statistic Reports

Report generation logic is based on three pillars:
- Data Provider
- Data Renderer
- Time Periods

Each of them provides the necessary data to generate and present statistical data from the marketplace and each of them can be easily extends.

## Data Provider
All available data providers are in the `MarketplaceSuite/src/Component/Statistic/ReportDataStrategy` directory.

Each provider must extend `AbstractReportDataProviderStrategy` and be tagged as `sylius.marketplace_suite.strategy.statistic_report.data` (this should be done automatically if you use autoconfigure).

In the data provider, you can set the owner of the report (e.g., Admin or User), and specify the type name, which is used in the support method or in getting dividers or translations.

## Data Renderer

Generated reports are presented by generic renderers. All default available renderers are in the `\Sylius\MarketplaceSuite\Component\Statistic\ReportDataRendererStrategy` directory. The specification of which renderer should process which data type is configured in Symfony parameters in `\Sylius\MarketplaceSuite\Component\Statistic\Resources\parameters.yaml`. You can easily override it to add new data types to existing renderers.

To create a custom data renderer, your class must implement `Sylius\MarketplaceSuite\Component\Statistic\ReportDataRendererStrategy\StatisticDataRendererInterface` and have methods implemented similarly to existing data renderers.

## Time Periods

By default, only monthly and weekly periods are available. To add a new custom time period, just add a new class that implements `\Sylius\MarketplaceSuite\Component\Statistic\PeriodStrategy\AbstractReportPeriodResolverStrategy` and add similar logic as in `\Sylius\MarketplaceSuite\Component\Statistic\PeriodStrategy\MonthlyReportPeriodResolver`. Reports are generated only when a report for a specific period doesn't exist in the database.

## Command
To systematically generate reports, please add the command `bin/console marketplace-suite:statistic:generate-reports` to your crontab. The command can be run as frequently as every 15 minutes because, under the hood, reports are generated only when declared periods without reports exist.
