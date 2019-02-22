# Salable Quantity Column in Product Grid Performance Proposal
This document provides a list of improvements and its motivation to the Salable Quantity column for the Product Grid in Magento Admin. This is a part of the [#2026](https://github.com/magento-engcom/msi/issues/2026) issue.

## Problem Statement
The `Magento\InventorySalesAdminUi\Ui\Component\Listing\Column\SalableQuantity` class is used to prepare the list of QTYs per given SKU. The result of the `SalableQuantity::prepareDataSource()` method execution is an updated list of parameters for the `salable_quantity` element for all `$dataSource['data']['items']` elements. Each of the `salable_quantity` dataset is loaded per SKU, which puts a huge pressure with high number of products in a Product Grid.

## Solution
The proposed solution is intended to solve the performance degradation issue with a high volume of `$dataSource['data']['items']` during Product Grid rendering.

## Inroduce new GetSalableQuantityData Class
The `\Magento\InventorySalesAdminUi\Model\GetSalableQuantityData` class should be responsible for returning a list of `salable_quantity` elements. The `execute()` method should receive an object of `\Magento\Framework\Api\SearchCriteriaInterface` with the predefined filter. The only filter will be added is a list of SKUs.

The `GetSalableQuantityData::execute()` method should be triggered for a list of SKUs allowed for Source Management (see `\Magento\InventoryConfigurationApi\Model\IsSourceItemManagementAllowedForProductTypeInterface`). As a result, only allowed SKUs will appear in the filter for the `execute()` method.

Option #1:
```
declare(strict_types=1);

namespace Magento\InventorySalesAdminUi\Model;

use Magento\Framework\Api\SearchCriteriaInterface;

class GetSalableQuantityData
{
    public function execute(SearchCriteriaInterface $searchCriteria): array
}
```

Alternatively, the `execute()` method annotation can include both a list of SKUs and allowed `type_id` list for Stock Management.

Option #2:
```
declare(strict_types=1);

namespace Magento\InventorySalesAdminUi\Model;

class GetSalableQuantityData
{
    public function execute(array $skus, array $typeIds): array
}
```

## The StockRepositoryInterface::getList() method usage
The `\Magento\InventorySalesAdminUi\Model\ResourceModel\GetAssignedStockIdsBySku` class is used to get a list of assigned stock ids per SKU. Later, Stock IDs are used to load Stock Item level configuration. The new `Magento\InventorySalesAdminUi\Model\GetSalableQuantityData::execute()` method implementation should load a key-value pair of stock item configuration for all SKUs provided.

As a result, the key-value pair will be similar to the following example:
```
[
    ‘sku1’ => [stock-item1-config…<\Magento\InventoryConfigurationApi\Api\Data\StockItemConfigurationInterface>]
    ‘sku2’ => [stock-item2-config...<\Magento\InventoryConfigurationApi\Api\Data\StockItemConfigurationInterface>]
]
```

## The StockRepositoryInterface::getList() method usage
The `Magento\InventorySalesAdminUi\Model\GetSalableQuantityData::execute()` method (alternative to the `\Magento\InventorySalesAdminUi\Model\GetSalableQuantityDataBySku::execute()` method) should use the `\Magento\InventoryApi\Api\StockRepositoryInterface::getList()` method in order to load stock for all assigned stocks per given SKU.

