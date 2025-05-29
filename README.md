# Subscription-LLD


Problem Statement : 


Design a subscription service, similar to mobile service providers postpaid connections
User Subscription and Service Usage:
Users can subscribe to service packs that offer access to various services (e.g., data, voice, SMS).
Users can avail services included within their subscribed packs.
Service Packs:
 Service packs may differ in terms of:
 Validity duration (e.g., monthly, quarterly)
 Usage limits (e.g., 100GB data, 1000 minutes)
 Pricing and rate plans (e.g., base cost, per-unit overage charges)
 Billing and Invoicing:
 The system should support:
 Billing at the time of pack purchase
 Billing for any over-usage or spill-over beyond pack limits


 Design approach

Using the bottom-up design approach. For this purpose, we will follow the steps below:
Identify and design the smallest components first, such as a feature .


---

### 📋 **Subscription System - Requirements**

---

#### ✅ **R1: Product Types**

* The subscription system should support **two product types**: **Prepaid** and **Postpaid**.

---

#### ✅ **R2: Offer and Variants**

* Each offer can have **multiple variants** (SKUs), and each variant provides access to different services such as **data**, **voice**, and **SMS**.

---

#### ✅ **R3: Commitment and Term**

* Variants must support **commitment terms**, such as **monthly** or **quarterly** subscriptions.

---

#### ✅ **R4: Feature Configuration**

* Each variant includes **features**, for example:

  * **Data Cap** (e.g., 100GB)
  * **Duration Cap Unit** (e.g., per day, per month)
  * **Over-Delegation**: A flag (`true`/`false`) indicating whether over-usage is allowed
  * **Charges Per Unit**: Defined in the feature set if over-delegation is enabled

---

#### ✅ **R5: Pricing**

* Each variant must have:

  * A **base cost**
  * **Per-unit overage charges**, applicable when the user exceeds the allocated limits

---

#### ✅ **R6: Admin Configuration**

* Admins should be able to **configure all metadata** including:

  * Products
  * Offers and variants
  * Commitment terms
  * Service caps and overage settings
  * Pricing

---

#### ✅ **R7: Customer Purchase**

* Customers can **browse and purchase** subscription products (Prepaid or Postpaid) with specific variants.

---

#### ✅ **R8: Overage Handling**

* If a customer exceeds the allowed usage:

  * And **over-delegation is enabled**, then the customer will be charged based on the **configured per-unit rate** for that feature (e.g., per GB/min/SMS).

---

#### ✅ **R9: Initial Billing**

* Billing is generated at the **time of subscription purchase**, based on:

  * The selected **plan**
  * Associated **offer and variant**



#### ✅ **R10: Overage Billing**

* **Additional billing** for any over-usage is generated **after the commitment period ends**, based on the actual usage and overage pricing.


### 👥 **Primary Actors**



#### 1. **Customer**

* **Description**: End-user who interacts with the system to subscribe to mobile service products.
* **Responsibilities**:

  * Browse available products and variants.
  * Purchase a subscription (prepaid or postpaid).
  * Use the subscribed services (data, voice, SMS).
  * Receive bills for subscription and over-usage.


#### 2. **Admin**

* **Description**: Authorized user responsible for configuring and maintaining the product catalog and pricing.
* **Responsibilities**:

  * Define and configure products (prepaid/postpaid).
  * Create offers and service pack variants.
  * Set usage caps, pricing, commitment terms, and overage charges.
  * Update configurations as needed.

---

#### 3. **System**

* **Description**: Automated backend system that enforces business rules and processes actions.
* **Responsibilities**:

  * Validate subscriptions and track usage in real-time.
  * Monitor usage against defined limits.
  * Automatically charge users for over-usage if applicable.
  * Generate billing invoices for both initial subscription and additional usage.



**Relationship Design for Subscription System**



🔷 **Generalization (Inheritance)**

                         Offer
                          ▲
               ┌──────────┴──────────┐
            Prepaid              Postpaid

                         SKU (Variant)
                          ▲
      ┌───────────────────┼───────────────────┐
   DataSKU            VoiceSKU              SMSSKU
   
   
Offer → Generalized into Prepaid and Postpaid

SKU → Generalized into DataSKU, VoiceSKU, and SMSSKU

Each SKU type can have specific attributes:

DataSKU: GB limit, speed

VoiceSKU: Minutes, call charges

SMSSKU: SMS count

🔷 Composition

SKU (Variant) ◼── has a ──▶ CommitmentTerm
Each SKU must define its commitment duration.

🔷 Aggregation

SKU ─◇── has ─▶ FeatureSet
FeatureSet:
   - Data Cap
   - Duration Cap Unit
   - Over-Delegation (ENUM)
   - Charges Per Unit


🔷 Enums

Enum CommitmentTerm:
   - DAILY
   - MONTHLY
   - YEARLY

Enum OverDelegation:
   - ENABLED
   - DISABLED
   - 
🔷 Associations

Offer ─────▶ * SKU
SKU ───────▶ Pricing
Billing ────▶ Invoice

🔷 Billing Composition

Billing ────┬────▶ Initial Billing
            └────▶ Overage Billing



**Use Case **

                        +--------------------+
                        |     Admin          |
                        +--------------------+
                           |          |
      +--------------------+          +--------------------+
      |                                         |
+------------+                        +-------------------------+
| Configure  |                        | Configure SKUs and      |
| Offers     |                        | Features (Data, SMS...) |
+------------+                        +-------------------------+
                                           |
                                           |
                                 +--------------------+
                                 | Set Pricing        |
                                 +--------------------+
                                           |
                                 +--------------------+
                                 | Set Commitment     |
                                 | Terms              |
                                 +--------------------+

                        +--------------------+
                        |     Customer       |
                        +--------------------+
                           |        |        |
                           |        |        |
        +------------------+        |        +------------------------+
        |                           |                                 |
+----------------+        +------------------+              +-----------------------+
| Browse Offers  |        | Subscribe to Pack|              | Use Services (Data,   |
+----------------+        +------------------+              | Voice, SMS)           |
                                  |                                 |
                                  |                                 |
                            +----------------+              +------------------------+
                            | Initial Billing|              |   Track Usage          |
                            +----------------+              +------------------------+
                                                                    |
                                                               +------------------+
                                                               | Check Over-Usage |
                                                               +------------------+
                                                                    |
                                                               +------------------+
                                                               | Charge Customer  |
                                                               +------------------+
                                                                    |
                                                               +------------------+
                                                               | Generate Invoice |
                                                               +------------------+

                             ^  
                             | 
                     +----------------+
                     |     System     |
                     +----------------+
                     
**Relationships**

Customer ─ Subscribe to Pack ──<<include>>── Initial Billing

Use Services ──<<include>>── Track Usage

Track Usage ──<<include>>── Check Over-Usage

Check Over-Usage ──<<extend>>── Charge Customer

Charge Customer ──<<include>>── Generate Invoice





Class Diagram 


Relationships Overview:
Product 1..* —> Offer

Offer 1..* —> Variant

Variant 1..* —> Feature

Customer 1..* —> Subscription

Subscription 1..* —> UsageRecord

Subscription 1..* —> Bill







**Activity Diagram**

![Activity](https://github.com/user-attachments/assets/f8fc0fd3-47b4-4b64-86a9-63449691b055)


This is not clear I took help from internet generate image based on my sketch data on my copy




### 📦 `Offer.java`


public abstract class Offer {
    private String id;
    private String name;
    private OfferType type;
}


### 📦 `OfferType.java`[restrict now enum we dont have aggregation here ]


public enum OfferType {
    PREPAID,
    POSTPAID
}


---

### 📦 `SKU.java`


public class SKU {
    private String id;
    private String name;
    private String validity;
    private Term commitment;
    private List<FeatureSet> features;
    private Pricing pricing;
}
```

### 📦 `Term.java`


public enum Term {
    MONTHLY,
    QUARTERLY,
    YEARLY
}




### 📦 `FeatureSet.java`


public abstract class FeatureSet {
    protected FeatureType type;
    protected String durationCap;
    protected boolean overDelegation;
    protected double chargesPerUnit;
}


### 📦 `FeatureType.java`


public enum FeatureType {
    DATA,
    VOICE,
    SMS
}




### 📦 `Pricing.java`


public class Pricing {
    private double baseCost;
    private double overagePerUnit;
}




### 📦 `Customer.java`


public class Customer {
    private String id;
    private String name;
    private List<Subscription> subscriptions;
}




### 📦 `Subscription.java`


public class Subscription {
    private String id;
    private String customerId;
    private String skuId;
    private LocalDate startDate;
    private LocalDate endDate;
    private UsageDetail usage;
}


### 📦 `UsageDetail.java`


public class UsageDetail {
    private double dataUsed;
    private double voiceUsed;
    private int smsUsed;
}


### 📦 `Billing.java`


public class Billing {
    private String id;
    private BillingType type;
    private double amount;
    private LocalDate date;
}


### 📦 `BillingType.java`


public enum BillingType {
    INITIAL,
    OVERAGE
}


---

### 📦 `Invoice.java`


public class Invoice {
    private String id;
    private String billingId;
    private LocalDate generatedDate;
}


---

### 📦 `Admin.java`


public class Admin {
    private String id;
    private String name;

    public void configure(Offer offer, SKU sku) {
        // logic to configure offer
    }
}



