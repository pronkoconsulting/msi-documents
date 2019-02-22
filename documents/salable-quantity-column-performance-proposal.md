# Salable Quantity Column in Product Grid Performance Proposal
This document provides a list of improvements and its motivation to the Salable Quantity column for the Product Grid in Magento Admin. This is a part of the [#2026](https://github.com/magento-engcom/msi/issues/2026) issue.

## Problem Statement
The `Magento\InventorySalesAdminUi\Ui\Component\Listing\Column\SalableQuantity` class is used to prepare the list of QTYs per given SKU. The result of the `SalableQuantity::prepareDataSource()` method execution is an updated list of parameters for the `salable_quantity` element for all `$dataSource['data']['items']` elements. Each of the `salable_quantity` dataset is loaded per SKU, which puts a huge pressure with high number of products in a Product Grid.
