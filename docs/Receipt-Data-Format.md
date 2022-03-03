# Receipt Data Format

Receipts may have varying layouts depending on the merchant, language, and geography. You can even format a date in a multitude of ways. 
We came up with a universal data model that focuses on the purpose of data in terms of financial management and accounting and not on the actual layout of the document itself.

## Receipt Data Fundamentals

1. **Item prices**. Apart from telling you how much each line item costs today (price on the price tag), a receipt can also tell you if this item is on sale and show you the original price (sometimes displayed as a strike-through). They can also show you the price after discounts and fees (what you’re actually paying for it).Finally, line item data on a receipt can display units of measure, number of units, prices per unit, and total amounts for a particular item. These amounts can be applied on the unit level as well as the line item level.

2. **Taxes**. Every amount you see on the receipt can be subject to taxes. That amount may already include taxes or not, depending on the country, merchant or both. Even when taxes are zero, it’s important to include the field because conceptually, any amount can be taxable. In some countries, even tips might be taxable.

3. **Discounts and fee**s. These can be applicable on both the item level and the receipt level. For example, on a grocery list, one of your items––say your paper towel––could be discounted (item level), or your entire shopping list is discounted (receipt level). The same goes for fees. You can have fees attached to a single item, like a recycling deposit for a can of beer, or for an entire purchase, like a delivery fee for an Uber Eats order.

4. **Gratuities**. Tips are listed separately because they’re applied after all the discounts/ fees are calculated, and are included in the totals on the receipt.

## Amount Structure

Every amount on the receipt is represented by the following structure. Let’s call it **The General `Amount` Structure**:

```typescript
class Amount {
    beforeTax: number,
    taxes: [
        {
            amount: number
        }
    ],
    afterTax: number
}
```

Usually only one of the fields `beforeTax` or `afterTax` will be populated. That’s because the amounts you see on the receipt might have taxes included depending on the country, merchant and/or type of item sold.

This structure also allows us to represent `taxes` in the context where they are actually applied. Note that `taxes` is an array - that’s because a receipt can have taxes split by type (e.g. provincial and federal taxes), which are presented separately, so for that amount the taxes array would hold two items.

Now, having determined the Amount Structure above, we can define the fields that represent what kind of amounts a receipt can carry.

```typescript
{
    items: [
        {
            unitOfMeasure: string,
            unitQuantity: number,
            unitListPrice: Amount,
            unitPrice: Amount,
            listPrice: Amount,
            priceBeforeDiscountsFees: Amount,
            discounts: Amount[],
            fees: Amount[],
            price: Amount
        }
    ],
    totalBeforeDiscountsFeesTips: Amount,
    discounts: Amount,
    fees: Amount,
    totalBeforeTips: Amount,
    tips: Amount,
    total: Amount

}
```

## Glossary of Receipt Fields

| Field                          | Type            | Description |
| --- | --- | --- |
| items                          | `array`           | If a receipt has item level info then it will be stored in this array of items |
| items.unitOfMeasure            | `string`          | Some receipt items have units of measure like kgs, litres etc. |
| items.unitQuantity             | `number`          | Usually if a unit of measure is present on an item it will also have the number of those units purchased |
| items.unitListPrice            | `Amount`          | List price of the unit of measure for this item in case it’s sold at a different price currently. You can sometimes see those striked through on a price tag |
| items.unitPrice                | `Amount`          | Price at which this item unit is currently sold at. Price on the price tag. |
| items.listPrice                | `Amount`          | List price of this line item in case it’s sold at a different price currently. Semantically should add up to (unitQuantity x unitListPrice). |
| items.priceBeforeDiscountsFees | `Amount`          | Price of this line item at which it is currently sold before any discounts and/or fees. Semantically should add up to (unitQuantity x unitPrice). |
| items.discounts                | `Array of Amount` | As the name suggests it’s item level discounts if applicable. |
| items.fees                     | `Array of Amount` | As the name suggests it’s item level fees if applicable. |
| items.price                    |                 | Price of this line item after all the discounts/fees were applied. This is the one you are actually paying for this item. Should add up to (items.priceBeforeDiscountsFees + items.discounts + items.fees)  |
| totalBeforeDiscountsFeesTips   | `Amount`          | Receipt amount before any discounts/fees. Semantically it should add up to the sum of<br>all items.priceBeforeDiscountsFees values OR if there is no item level information on the receipt it can be an independent value just denoting the amount on the receipt before any discounts/fees. |
| discounts                      | `Array of Amount` | Receipt level discounts if applicable. |
| fees                           | `Array of Amount` | Receipt level fees if applicable. |
| totalBeforeTips                | `Amount`          | Receipt amount after all the discounts/fees but before any tips/gratuities. Semantically it should add up to the sum of all items.price values OR if there is no item level information on the receipt it can be an independent value just denoting the amount on the receipt after all the discounts/fees. It should also add up to (totalBeforeDiscountsFeesTips + discounts + fees). |
| tips                           | `Array of Amount` | Any tips or gratuities if applicable. |
| total                          | `Amount`          | Total amount of the receipt after all the tips/gratuities applied. It should add up to totalBeforeTips + tips.  |

## Data Model Examples

Let's consider some common examples of receipt layouts and how they map to the described structure. 

### Totals

---

#### Exhibit A

Here, we have a standard North American receipt with a before-tax subtotal, taxes and after-tax total.

![Exhibit A](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/receipts/receipt-example-totals-exhibit-a.png)

```json
{
    "totalBeforeDiscountsFeesTips": {
        "beforeTax": 79.99,
        "taxes": [
            {
                "amount": 10.4,
                "type": "HST"
            }
        ],
        "afterTax": 90.39
    },
    "totalBeforeTips": {
        "beforeTax": 79.99,
        "taxes": [
            {
                "amount": 10.4,
                "type": "HST"
            }
        ],
        "afterTax": 90.39
    },
    "total": {
        "beforeTax": 79.99,
        "taxes": [
            {
                "amount": 10.4,
                "type": "HST"
            }
        ],
        "afterTax": 90.39
    }
}
```

---

#### Exhibit B

In this example, we have a standard receipt from European countries that have the VAT taxes included in the final price. The taxes are listed underneath the final total.

![Exhibit B](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/receipts/receipt-example-totals-exhibit-b.jpg)

```json
{
    "totalBeforeDiscountsFeesTips": {
        "afterTax": 29.45,
        "taxes": [
            {
                "amount": 0.78,
                "type": "Sales Tax"
            }
        ]
    },
    "totalBeforeTips": {
        "afterTax": 29.45,
        "taxes": [
            {
                "amount": 0.78,
                "type": "Sales Tax"
            }
        ]
    },
    "total": {
        "taxes": [
            {
                "amount": 0.78,
                "type": "Sales Tax"
            }
        ],
        "afterTax": 29.45
    }
}
```


---

#### Exhibit C

Here is where it gets even more complicated. In this example we have a North American restaurant receipt with a subtotal of all the items, the total after taxes, followed by a tip, and then the final payment amount.

![Exhibit C](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/receipts/receipt-example-totals-exhibit-c.jpg)

```json
{
    "totalBeforeDiscountsFeesTips": {
        "beforeTax": 16.65,
        "afterTax": 18.81,
        "taxes": [
            {
                "amount": 2.16,
                "type": "HST"
            }
        ]
    },
    "totalBeforeTips": {
        "beforeTax": 16.65,
        "afterTax": 18.81,
        "taxes": [
            {
                "amount": 2.16,
                "type": "HST"
            }
        ]
    },
    "tips": [
        {
            "beforeTax": 1,
            "taxes": [
                {
                    "amount": 0
                }
            ],
            "afterTax": 1
        }
    ],
    "total": {
        "beforeTax": 17.65,
        "taxes": [
            {
                "amount": 2.16,
                "type": "HST"
            },
            {
                "amount": 0
            }
        ],
        "afterTax": 19.81
    }
}
```

---

#### Exhibit D

In this example we have a receipt with free shipping that displays the shipping charges, and then the same amount as a discount.

![Exhibit D](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/receipts/receipt-example-totals-exhibit-d.jpg)

```json
{
    "totalBeforeDiscountsFeesTips": {
        "beforeTax": 91.7
    },
    "discounts": [
        {
            "beforeTax": -4.95,
            "type": "general"
        }
    ],
    "fees": [
        {
            "beforeTax": 4.95,
            "description": "SHIPPING",
            "type": "shipping"
        }
    ],
    "totalBeforeTips": {
        "afterTax": 96.29,
        "taxes": [
            {
                "amount": 4.59,
                "type": "HST"
            }
        ]
    },
    "total": {
        "taxes": [
            {
                "amount": 4.59,
                "type": "HST"
            }
        ],
        "afterTax": 96.29
    }
}
```

---

#### Exhibit E

Finally, we have a payment receipt where the tip was applied at the end.

![Exhibit E](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/receipts/receipt-example-totals-exhibit-e.jpg)

```json
{
    "totalBeforeDiscountsFeesTips": {
        "beforeTax": 15.8
    },
    "totalBeforeTips": {
        "beforeTax": 15.8
    },
    "tips": [
        {
            "beforeTax": 2.37,
            "taxes": [
                {
                    "amount": 0
                }
            ],
            "afterTax": 2.37
        }
    ],
    "total": {
        "beforeTax": 18.17,
        "taxes": [
            {
                "amount": 0
            }
        ],
        "afterTax": 18.17
    }
}
```

### Items

---

#### Exhibit A

In this example from a North American clothing store receipt, we have:
The original price before tax, along with
the item quantity final price,
and the discount amount.

**Note**: *These prices are listed before taxes.*

![Exhibit A](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/receipts/receipt-example-items-exhibit-a.png)

```json
{
    "items": [
        {
            "unitListPrice": {
                "beforeTax": 20
            },
            "unitPrice": {
                "beforeTax": 15
            },
            "priceBeforeDiscountsFees": {
                "beforeTax": 20
            },
            "price": {
                "beforeTax": 15
            },
            "discounts": [
                {
                    "description": "Promotional Sale",
                    "beforeTax": -5,
                    "type": "general"
                }
            ],
            "itemIDs": [
                "34152421"
            ],
            "name": "BOO Cl IN STNDRD FT VNK",
            "unitQuantity": 1
        }
    ]
}

```

---

#### Exhibit B

In this example from a North American grocery store, we have the original list price and then the final price taking into account the unit quantity.


![Exhibit B](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/receipts/receipt-example-items-exhibit-b.png)

```json
{
    "items": [
        {
            "unitListPrice": {
                "beforeTax": 4.34
            },
            "price": {
                "beforeTax": 4.27
            },
            "itemIDs": [
                "3438"
            ],
            "name": "APPLE AMBROSIA R",
            "unitOfMeasure": "kg",
            "unitQuantity": 0.985
        }
    ]
}

```

---

#### Exhibit C

In this example from a UK grocery receipt, the taxes are included in the price so the prices are listed as after tax.

![Exhibit C](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/receipts/receipt-example-items-exhibit-c.png)

```json
{
    "priceBeforeDiscountsFees": {
        "afterTax": 2.25
    },
    "price": {
        "afterTax": 2.25
    },
    "name": "ST D STRAWB SPREAD",
    "unitQuantity": 1
}

```


