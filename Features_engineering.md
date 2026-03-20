Features engineering
The general principle to follow is that features should describe the customer’s behavior profile before they became inactive, not after. Features must be computed strictly from the pre-cutoff period and excludes anything from the outcome window, to avoid leakage.
Feature groups are: 
-covering monetary behavior (Monetary features)
    These features try to measure the economic value and purchase intensity of the customer before inactivity, like: total lifetime spend, average order value, standard deviation of order value, highest single purchase amount, spend in the last 6/12 months before inactivity, and spend trend.
    - TOTAL_NET_VALID_PRE
    - AVG_ORDER_VALUE_VALID_PRE
    - STD_ORDER_VALUE_VALID_PRE
    - MAX_ORDER_VALUE_VALID_PRE
    - SPEND_LAST_6M_BEFORE_LAST_PURCHASE
    - SPEND_LAST_12M_BEFORE_LAST_PURCHASE
    - SPEND_TREND_SLOPE_PRE

    The logic is simple: customers who historically spent more, placed larger orders, or showed stronger recent spending before disappearing are often more commercially relevant and may be more worth recovering. At the same time, variability in spend can matter: a customer with stable ordering behavior may be different from one with sporadic, one-off large purchases. The spend-trend feature is especially useful because it tries to distinguish customers who were already declining before inactivity from customers who were still stable or growing when they stopped buying.

    A deliberate modeling choice was to compute the recent-spend windows relative to the customer’s last observed purchase, not just relative to the cutoff. For already inactive customers, “last 6 months before cutoff” would often be zero by definition, so anchoring the window to the last purchase preserves information about what the customer looked like right before disengaging. That is consistent with "capture behavior before going inactive.”

-frequency/recency
    These features capture how often the customer bought and how recently they were engaged. The roadmap explicitly lists total number of purchase months, purchase frequency per active month, months since last purchase, average gap between purchases, and customer tenure from registration to last purchase.
    N_VALID_TXNS_PRE
    N_ACTIVE_MONTHS_PRE
    PURCHASE_FREQUENCY_PRE
    MONTHS_SINCE_LAST_PURCHASE_AT_CUTOFF
    AVG_GAP_MONTHS_PRE
    CUSTOMER_TENURE_TO_LAST_PURCHASE_MONTHS
    CLIENT_AGE_AT_CUTOFF_MONTHS

    These are important because reactivation is not only about value; it is also about relationship shape. A customer who bought frequently over many months is different from a customer who made one or two isolated purchases. Similarly, the average gap between purchase months can tell us whether the customer had a steady replenishment pattern or a very irregular one. Tenure-related features matter because long-established customers may be easier to win back than newly registered customers who never really developed into habitual buyers.

    Recency is especially central in CRM. The roadmap frames inactivity itself around time since last purchase, so MONTHS_SINCE_LAST_PURCHASE_AT_CUTOFF is not just another feature; it is closely related to the business definition of inactivity. Even within the already-inactive cohort, someone inactive for 24 months may behave very differently from someone inactive for 40 or 50 months

-product and channel mix
    These features aim to describe what the customer buys and how they buy it. The roadmap recommends tool-vs-parts ratio, number of distinct product families and groups, dominant sales channel, number of channels used, and a product-diversity index such as entropy on FAMILY_CODE.
    In the notebook, these became:
    TOOL_RATIO_PRE
    DISTINCT_FAMILY_COUNT_PRE
    DISTINCT_GROUP_COUNT_PRE
    DISTINCT_CHANNEL_COUNT_PRE
    DOMINANT_CHANNEL_PRE
    FAMILY_ENTROPY_PRE

    These features matter because not all customers relate to the business in the same way. A customer buying only one narrow family of products may be more replaceable or price-sensitive than a customer buying across many families. A broad product mix can indicate stronger integration into the firm’s assortment and potentially greater reactivation potential. Likewise, channel behavior can signal commercial style: some customers are highly concentrated in one channel, while others are more flexible and multi-channel. The roadmap treats these as important behavioral descriptors because they capture relationship depth and purchasing structure beyond pure spend.

-client registry attributes
    These are the static or semi-static business descriptors available in TOOL_CLIENT. 
    In the notebook, these became:
    ECONOMIC_POT
    ECO_POT_CLASS
    RISK_CAT
    N_EMPLOYEES
    TRADE_SECTOR
    REGION_FILLED
    CLIENT_AGE_AT_CUTOFF_MONTHS

    These features are important because reactivation is not only a behavioral pattern but also a business-segmentation problem. A larger company may have more stable replenishment needs. A higher ECONOMIC_POT customer is commercially more attractive and may also behave differently from low-potential accounts. Trade sector and region can capture structural differences in buying cycles, local market conditions, or sector-specific demand patterns. The roadmap highlights region specifically as a possible geographic signal and economic potential as one of the most important business variables.

-behavioral signals
    This group was chosen to model not only what customers buy, but also how clean or problematic their order history is. (cancellation rate, return rate, and seasonality pattern as behavioral signals)

    CANCELLATION_RATE_PRE
    RETURN_RATE_PRE
    TXN_AUGUST_SHARE_PRE
    ACTIVE_MONTH_AUGUST_SHARE_PRE

    These features matter because a customer with a history of cancelled orders or many returns may be commercially unstable or operationally difficult, which could reduce the likelihood or value of reactivation. The roadmap specifically recommends excluding cancelled orders from feature engineering but still optionally using cancellation rate as a feature, which is exactly the logic applied in the notebook. Likewise, returns correspond to negative NET values, and the roadmap explicitly says this requires an intentional handling choice.

    Seasonality was included because the roadmap highlights a consistent August drop in transactions and warns that some inactivity may simply reflect normal seasonal behavior rather than churn. A customer who systematically buys in certain months should not be interpreted in the same way as a customer whose behavior is erratic or declining. The August-share features are a compact way to give the model some awareness of seasonality without overcomplicating the design.


Taken together, the selected features answer five complementary questions about each customer:

How valuable were they?
Monetary features answer this.

How regular and recent was the relationship?
Frequency and recency features answer this.

How broad or narrow was their buying behavior?
Product and channel features answer this.

What kind of customer are they structurally?
Registry features answer this.

Was their history clean and stable, or noisy and problematic?
Behavioral signals answer this.