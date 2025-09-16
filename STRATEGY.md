# GTM Traffic Source Strategy & Decision Logic

## Executive Summary

This document outlines the strategic approach and decision logic for the GTM Traffic Source classification system. The solution prioritizes **technical accuracy** while maintaining **marketing flexibility** through intelligent plausibility checks.

## 🎯 Strategic Objectives

### Primary Goals
1. **Accurate Attribution** - Correctly identify traffic sources
2. **Technical Reliability** - Minimize false classifications
3. **Marketing Flexibility** - Allow precise campaign control
4. **Data Quality** - Ensure consistent, trustworthy analytics

### Key Challenges
- **GCLID "Carry-over"** - Click IDs can persist in URLs across sessions
- **UTM Errors** - Manual UTM parameters can be incorrectly set
- **Conflicting Signals** - Multiple parameters may suggest different sources
- **Cross-Platform Attribution** - Users may visit multiple platforms before converting

## 🏗️ Core Principles

### **Hybrid Attribution Model**
The system combines **technical signals** (Click IDs, Referrers) with **marketing signals** (UTM parameters) while applying **plausibility checks** to prevent false attributions.

### **Channel Group Abstraction**
The system uses **Channel Groups** as an abstraction layer to simplify UTM validation and prevent error-prone, granular scenario mapping. This ensures consistent behavior across all Click IDs within the same traffic type.

### **Priority-Based Classification**
Traffic sources are classified using a strict priority hierarchy that prioritizes technical accuracy while allowing marketing control through plausibility checks.

## 🔍 Classification Logic Framework

### 1. **Technical Signals (High Priority)**

#### **Click IDs - Automated & Reliable**
| Parameter | Platform | Reliability | Use Case |
|-----------|----------|-------------|----------|
| `gclid` | Google Ads | 99% | `paid_search` |
| `gbraid` | Google Ads | 99% | `paid_search` (Enhanced Conversions) |
| `wbraid` | Google Ads | 99% | `paid_search` (Enhanced Conversions) |
| `msclkid` | Microsoft Ads | 99% | `paid_search` |
| `fbclid` | Facebook Ads | 99% | `paid_social` |
| `ttclid` | TikTok Ads | 99% | `paid_social` |
| `li_fat_id` | LinkedIn Ads | 99% | `paid_social` |
| `twclid` | Twitter Ads | 99% | `paid_social` |

**Advantages:**
- ✅ Automatically set by platforms
- ✅ Technically accurate
- ✅ Cannot be manually manipulated
- ✅ Used to validate other signals

### 2. **Marketing Signals (Medium Priority)**

#### **UTM Parameters - Manual & Flexible**
| Parameter | Purpose | Control Level | Error Probability | Reasoning |
|-----------|---------|---------------|-------------------|-----------|
| `utm_source` | Traffic source | High | High | *Different teams use different naming conventions (google vs Google vs Google Ads)* |
| `utm_medium` | Channel type | High | High | *Inconsistent standards across teams (cpc vs CPC vs paid_search)* |
| `utm_campaign` | Campaign name | High | Medium | *Marketing teams know their campaigns, but may use different formats* |
| `utm_term` | Search term | High | Low | *Usually auto-generated or clearly defined by search platforms* |
| `utm_content` | Content variant | High | Low | *Typically auto-generated or clearly defined by ad platforms* |

**Advantages:**
- ✅ Full marketing control
- ✅ Precise campaign tracking
- ✅ Custom naming conventions
- ✅ Detailed attribution

**Risks:**
- ⚠️ Manual entry errors
- ⚠️ Inconsistent naming
- ⚠️ Can conflict with technical signals
- ⚠️ May be set incorrectly

### 3. **Organic Signals (Lower Priority - Exclusion Logic)**

#### **Search Engines & Social Platforms**
- **`organic_search`** - Google, Bing, Yahoo, etc.
- **`organic_social`** - Facebook, Instagram, LinkedIn, etc.
- **`organic_shopping`** - Amazon, Google Shopping, etc.
- **`organic_video`** - YouTube, Vimeo, etc.

**Note:** Organic signals have lower priority due to **exclusion logic** - we must first check what it's NOT (advertising/marketing) before determining what it IS (organic). This ensures accurate attribution by ruling out paid traffic first.

## 📊 Channel Groups Reference

The system uses **Channel Groups** as an abstraction layer to simplify UTM validation and prevent error-prone, granular scenario mapping. This ensures consistent behavior across all Click IDs within the same traffic type.

### **Complete Channel Groups List**

| **Channel Group** | **Description** | **Typical Sources** | **Typical Mediums** |
|-------------------|-----------------|---------------------|---------------------|
| `paid_search` | Search engine advertising | Google, Bing, Yahoo | `cpc`, `paid_search` |
| `paid_social` | Social media advertising | Facebook, Instagram, LinkedIn, TikTok, Twitter | `paid_social`, `social` |
| `paid_shopping` | Shopping platform advertising | Amazon, Google Shopping, eBay | `cpc`, `shopping` |
| `paid_video` | Video platform advertising | YouTube, Vimeo, TikTok | `cpc`, `video` |
| `display` | Display advertising | Various ad networks | `display`, `banner` |
| `display_remarketing` | Remarketing/retargeting | Criteo, Google Display | `remarketing`, `retargeting` |
| `organic_search` | Organic search traffic | Google, Bing, Yahoo, DuckDuckGo | `organic` |
| `organic_social` | Organic social media | Facebook, Instagram, LinkedIn | `social` |
| `organic_video` | Organic video content | YouTube, Vimeo, Twitch | `video` |
| `organic_shopping` | Organic shopping traffic | Amazon, Google Shopping | `organic` |
| `email` | Email marketing | Mail clients, email campaigns | `email` |
| `affiliate` | Affiliate marketing | Partner websites | `affiliate` |
| `referral` | Direct referral traffic | External websites | `referral` |
| `direct` | Direct traffic | No referrer | `(none)` |
| `unassigned` | Unclassified traffic | Unknown sources | `unknown` |

## 🌳 Traffic Source Classification Decision Tree

The following comprehensive decision tree shows all possible scenarios and their outcomes:

```
🔄 TRAFFIC SOURCE CLASSIFICATION
    │
    ├─ 1️⃣ CLICK IDS (Highest Priority)
    │   ├─ gclid → Google Ads (paid_search)
    │   │   ├─ Channel Group: paid_search
    │   │   ├─ Source: google
    │   │   ├─ Medium: cpc
    │   │   ├─ Ad Platform: google_ads
    │   │   └─ UTM Override Check:
    │   │       ├─ utm_source plausible? → Override source
    │   │       ├─ utm_medium NOT overridden (Click ID is source of truth)
    │   │       └─ utm_campaign/term/content → Always override
    │   │
    │   ├─ fbclid → Facebook Ads (paid_social)
    │   │   ├─ Channel Group: paid_social
    │   │   ├─ Source: facebook
    │   │   ├─ Medium: paid_social
    │   │   ├─ Ad Platform: facebook_ads
    │   │   └─ UTM Override Check: (same logic)
    │   │
    │   ├─ msclkid → Microsoft Ads (paid_search)
    │   │   ├─ Channel Group: paid_search
    │   │   ├─ Source: microsoft
    │   │   ├─ Medium: cpc
    │   │   └─ Ad Platform: microsoft_ads
    │   │
    │   ├─ ttclid → TikTok Ads (paid_social)
    │   │   ├─ Channel Group: paid_social
    │   │   ├─ Source: tiktok
    │   │   ├─ Medium: paid_social
    │   │   └─ Ad Platform: tiktok_ads
    │   │
    │   ├─ li_fat_id → LinkedIn Ads (paid_social)
    │   │   ├─ Channel Group: paid_social
    │   │   ├─ Source: linkedin
    │   │   ├─ Medium: paid_social
    │   │   └─ Ad Platform: linkedin_ads
    │   │
    │   ├─ gbraid → Google Ads Enhanced (paid_search)
    │   │   ├─ Channel Group: paid_search
    │   │   ├─ Source: google
    │   │   ├─ Medium: cpc
    │   │   └─ Ad Platform: google_ads
    │   │
    │   ├─ wbraid → Google Ads Enhanced (paid_search)
    │   │   ├─ Channel Group: paid_search
    │   │   ├─ Source: google
    │   │   ├─ Medium: cpc
    │   │   └─ Ad Platform: google_ads
    │   │
    │   └─ twclid → Twitter Ads (paid_social)
    │       ├─ Channel Group: paid_social
    │       ├─ Source: twitter
    │       ├─ Medium: paid_social
    │       └─ Ad Platform: twitter_ads
    │
    ├─ 2️⃣ UTM PARAMETERS (If NO Click ID)
    │   ├─ utm_source present?
    │   │   ├─ ✅ Yes → Classify by utm_medium
    │   │   │   ├─ medium = "cpc/ppc" → Check source for channel group:
    │   │   │   │   ├─ Search engines → paid_search
    │   │   │   │   ├─ Social platforms → paid_social
    │   │   │   │   ├─ Shopping platforms → paid_shopping
    │   │   │   │   └─ Video platforms → paid_video
    │   │   │   │
    │   │   │   ├─ medium = "display/banner" → display/display_remarketing
    │   │   │   ├─ medium = "email" → email
    │   │   │   ├─ medium = "affiliate" → affiliate
    │   │   │   ├─ medium = "social" → organic_social
    │   │   │   ├─ medium = "video" → organic_video
    │   │   │   └─ Other → unassigned
    │   │   │
    │   │   └─ ❌ No → Continue to 3️⃣
    │
    ├─ 3️⃣ ORGANIC SEARCH
    │   ├─ Referrer = Search Engine? (Google, Bing, Yahoo, etc.)
    │   │   ├─ ✅ Yes → organic_search
    │   │   │   ├─ Google → source = "google"
    │   │   │   ├─ Bing → source = "bing"
    │   │   │   ├─ Yahoo → source = "yahoo"
    │   │   │   ├─ DuckDuckGo → source = "duckduckgo"
    │   │   │   ├─ Yandex → source = "yandex"
    │   │   │   ├─ Baidu → source = "baidu"
    │   │   │   └─ Other → source = referrer domain
    │   │   │
    │   │   └─ ❌ No → Continue to 4️⃣
    │
    ├─ 4️⃣ SOCIAL MEDIA
    │   ├─ Referrer = Social Platform? (Facebook, Instagram, etc.)
    │   │   ├─ ✅ Yes → organic_social
    │   │   │   ├─ Facebook → source = "facebook"
    │   │   │   ├─ Instagram → source = "instagram"
    │   │   │   ├─ LinkedIn → source = "linkedin"
    │   │   │   ├─ TikTok → source = "tiktok"
    │   │   │   ├─ Twitter → source = "twitter"
    │   │   │   ├─ YouTube → source = "youtube" (organic_video)
    │   │   │   └─ Other → source = referrer domain
    │   │   │
    │   │   └─ ❌ No → Continue to 5️⃣
    │
    ├─ 5️⃣ MAIL CLIENTS
    │   ├─ Referrer = Mail Client? (Gmail, Outlook, etc.)
    │   │   ├─ ✅ Yes → email
    │   │   │   ├─ Source = "mail"
    │   │   │   └─ Medium = "email"
    │   │   │
    │   │   └─ ❌ No → Continue to 6️⃣
    │
    ├─ 6️⃣ REFERRAL
    │   ├─ Referrer present AND not Self-Referral?
    │   │   ├─ ✅ Yes → referral
    │   │   │   ├─ Source = referrer domain
    │   │   │   └─ Medium = "referral"
    │   │   │
    │   │   └─ ❌ No → Continue to 7️⃣
    │
    └─ 7️⃣ DIRECT
        └─ No referrer → direct
            ├─ Source = "(direct)"
            └─ Medium = "(none)"
```

### **Key Decision Points:**

1. **Click ID Detection** - Highest priority, determines channel group
2. **UTM Validation** - Only if no Click ID, or as override for Click ID
3. **Referrer Analysis** - Ground truth for organic traffic classification
4. **Self-Referral Check** - Prevents internal traffic from being classified as referral
5. **Debug Tool Exclusion** - GTM Preview Mode, etc. excluded from organic classification

### **Plausibility Check Logic:**
- **Click ID + UTM:** UTM can override source/medium only if plausible for the channel group
- **UTM Only:** Channel group determined by source + medium combination
- **Organic Only:** Referrer analysis determines source and channel group
- **Fallback:** Direct traffic when no other signals present

## ⚠️ Conflict Detection & Plausibility Checks

### **The Problem: Technical vs. Marketing Signals**

**Click IDs** (technical signals) and **UTM parameters** (marketing signals) can contradict each other. The system intelligently resolves these conflicts:

#### **Example Conflict:**
```
URL: ?gclid=123&utm_source=facebook&utm_medium=social
Click ID says: "Google Ads" (paid_search)
UTM says: "Facebook Social" (paid_social)
```

**Solution:** Click ID has priority, but UTM is checked for plausibility.

## 🎯 Plausibility Check Matrix

### **Click ID + UTM Conflicts**

| **Click ID** | **UTM Source** | **UTM Medium** | **Plausible?** | **Final Source** | **Final Medium** | **Conflict** |
|--------------|----------------|----------------|----------------|-------------------|-------------------|--------------|
| **Google Ads (gclid)** | `google` | `cpc` | ✅ **Yes** | `google` | `cpc` | - |
| **Google Ads (gclid)** | `facebook` | `social` | ❌ **No** | `google` | `cpc` | `source_conflict` |
| **Google Ads (gclid)** | `google` | `social` | ❌ **No** | `google` | `cpc` | `medium_conflict` |
| **Google Ads (gclid)** | `bing` | `cpc` | ✅ **Yes** | `bing` | `cpc` | - |
| **Facebook Ads (fbclid)** | `facebook` | `paid_social` | ✅ **Yes** | `facebook` | `paid_social` | - |
| **Facebook Ads (fbclid)** | `google` | `cpc` | ❌ **No** | `facebook` | `paid_social` | `source_conflict` |
| **Facebook Ads (fbclid)** | `facebook` | `organic` | ❌ **No** | `facebook` | `paid_social` | `medium_conflict` |
| **Microsoft Ads (msclkid)** | `microsoft` | `cpc` | ✅ **Yes** | `microsoft` | `cpc` | - |
| **Microsoft Ads (msclkid)** | `linkedin` | `social` | ❌ **No** | `microsoft` | `cpc` | `source_conflict` |
| **TikTok Ads (ttclid)** | `tiktok` | `paid_social` | ✅ **Yes** | `tiktok` | `paid_social` | - |
| **TikTok Ads (ttclid)** | `google` | `cpc` | ❌ **No** | `tiktok` | `paid_social` | `source_conflict` |
| **LinkedIn Ads (li_fat_id)** | `linkedin` | `paid_social` | ✅ **Yes** | `linkedin` | `paid_social` | - |
| **LinkedIn Ads (li_fat_id)** | `facebook` | `social` | ❌ **No** | `linkedin` | `paid_social` | `source_conflict` |

### **Channel Group Plausibility Rules**

#### **🔍 `paid_search` (gclid, gbraid, wbraid, msclkid)**

**When a Click ID indicates Paid Search, these UTM parameters are checked for plausibility:**

| **UTM Parameter Type** | **Plausible Values** | **Examples** | **Why Plausible** |
|------------------------|----------------------|--------------|-------------------|
| **utm_source** | Search Engines | `google`, `bing`, `yahoo`, `microsoft` | These are actual search engines that can generate paid search traffic |
| **utm_source** | ❌ Social Platforms | `facebook`, `instagram`, `linkedin` | These are not search engines |
| **utm_medium** | CPC/PPC Terms | `cpc`, `ppc`, `paid_search` | Standard paid search medium values |
| **utm_medium** | ❌ Other Channels | `social`, `display`, `email` | These represent different channel types |

**Example Logic:**
- `gclid=123&utm_source=google&utm_medium=cpc` → ✅ **Plausible** (search engine + CPC)
- `gclid=123&utm_source=facebook&utm_medium=social` → ❌ **Not Plausible** (social platform + social medium)

#### **📱 `paid_social` (fbclid, ttclid, li_fat_id, twclid)**

**When a Click ID indicates Paid Social, these UTM parameters are checked for plausibility:**

| **UTM Parameter Type** | **Plausible Values** | **Examples** | **Why Plausible** |
|------------------------|----------------------|--------------|-------------------|
| **utm_source** | Social Platforms | `facebook`, `instagram`, `linkedin`, `tiktok` | These are actual social platforms that can generate paid social traffic |
| **utm_source** | ❌ Search Engines | `google`, `bing`, `yahoo` | These are not social platforms |
| **utm_medium** | Social/CPC Terms | `social`, `cpc`, `social_media` | Standard paid social medium values |
| **utm_medium** | ❌ Search Terms | `organic`, `ppc` | These represent different channel types |

**Example Logic:**
- `fbclid=456&utm_source=facebook&utm_medium=cpc` → ✅ **Plausible** (social platform + CPC)
- `fbclid=456&utm_source=google&utm_medium=organic` → ❌ **Not Plausible** (search engine + organic)

#### **🛒 `paid_shopping` (Amazon, Google Shopping)**

**When UTM parameters indicate Paid Shopping, these combinations are valid:**

| **UTM Parameter Type** | **Plausible Values** | **Examples** | **Why Plausible** |
|------------------------|----------------------|--------------|-------------------|
| **utm_source** | Shopping Platforms | `amazon`, `google`, `shopify`, `ebay` | These are actual shopping platforms that can generate paid shopping traffic |
| **utm_source** | ❌ Social Platforms | `facebook`, `linkedin`, `tiktok` | These are not shopping platforms |
| **utm_medium** | Shopping/CPC Terms | `cpc`, `shopping`, `product_ads` | Standard paid shopping medium values |
| **utm_medium** | ❌ Other Channels | `social`, `email`, `organic` | These represent different channel types |

**Example Logic:**
- `utm_source=amazon&utm_medium=cpc` → ✅ **Plausible** (shopping platform + CPC)
- `utm_source=facebook&utm_medium=shopping` → ❌ **Not Plausible** (social platform + shopping)

#### **📺 Display/Remarketing**

**When UTM parameters indicate Display/Remarketing, these combinations are valid:**

| **UTM Parameter Type** | **Plausible Values** | **Examples** | **Why Plausible** |
|------------------------|----------------------|--------------|-------------------|
| **utm_source** | Display Networks | `criteo`, `adnxs`, `google` | These are actual display networks that can generate display traffic |
| **utm_source** | ❌ Search Engines (as CPC) | `google`, `bing` (when medium=cpc) | Would be confused with search |
| **utm_medium** | Display Terms | `display`, `banner`, `remarketing` | Standard display medium values |
| **utm_medium** | ❌ Search Terms | `cpc`, `ppc` | Would be confused with search |

**Example Logic:**
- `utm_source=criteo&utm_medium=display` → ✅ **Plausible** (display network + display)
- `utm_source=google&utm_medium=cpc` → ❌ **Not Plausible** (would be classified as paid search)

## 🔧 Conflict Resolution in Practice

### **Scenario 1: Compatible UTM (No Conflict)**
```
URL: ?gclid=123&utm_source=google&utm_medium=cpc&utm_campaign=winter2024
Result:
✅ Click ID: Google Ads (paid_search)
✅ UTM Source: google (plausible for paid_search)
✅ UTM Medium: cpc (plausible for paid_search)
✅ Final: source=google, medium=cpc, campaign=winter2024
```

### **Scenario 2: Incompatible UTM (Conflict Detected)**
```
URL: ?gclid=123&utm_source=facebook&utm_medium=social&utm_campaign=winter2024
Result:
✅ Click ID: Google Ads (paid_search)
❌ UTM Source: facebook (not plausible for paid_search)
❌ UTM Medium: social (not plausible for paid_search)
✅ Final: source=google, medium=cpc, campaign=winter2024
⚠️ Conflict: source_conflict|medium_conflict
```

### **Scenario 3: Partially Compatible (Mixed Conflict)**
```
URL: ?fbclid=456&utm_source=facebook&utm_medium=organic&utm_campaign=summer2024
Result:
✅ Click ID: Facebook Ads (paid_social)
✅ UTM Source: facebook (plausible for paid_social)
❌ UTM Medium: organic (not plausible for paid_social)
✅ Final: source=facebook, medium=cpc, campaign=summer2024
⚠️ Conflict: medium_conflict
```

## 📊 UTM-Only Classification (Without Click ID)

### **Channel Group Determination by UTM**

| **UTM Source** | **UTM Medium** | **Channel Group** | **Final Source** | **Final Medium** | **Reason** |
|----------------|----------------|-------------------|-------------------|-------------------|------------|
| `google` | `cpc` | `paid_search` | `google` | `cpc` | Search Engine + CPC |
| `facebook` | `social` | `organic_social` | `facebook` | `social` | Social Platform + Social |
| `facebook` | `cpc` | `paid_social` | `facebook` | `cpc` | Social Platform + CPC |
| `amazon` | `cpc` | `paid_shopping` | `amazon` | `cpc` | Shopping Platform + CPC |
| `criteo` | `display` | `display` | `criteo` | `display` | Display Network + Display |
| `newsletter` | `email` | `email` | `newsletter` | `email` | Email Source + Email |
| `mail` | `(none)` | `email` | `mail` | `email` | Mail client referrer |
| `partner` | `affiliate` | `affiliate` | `partner` | `affiliate` | Partner + Affiliate |

### **Source-Only Fallback (No UTM Medium)**

When only `utm_source` is provided (common marketing scenario), the system applies intelligent fallback classification:

| **UTM Source** | **Fallback Classification** | **Channel Group** | **Final Medium** | **Reason** |
|----------------|----------------------------|-------------------|-------------------|------------|
| `facebook` | Social Platform | `organic_social` | `social` | Social source without medium |
| `instagram` | Social Platform | `organic_social` | `social` | Social source without medium |
| `linkedin` | Social Platform | `organic_social` | `social` | Social source without medium |
| `youtube` | Video Platform | `organic_video` | `video` | Video source without medium |
| `vimeo` | Video Platform | `organic_video` | `video` | Video source without medium |
| `google` | Search Engine | `organic_search` | `organic` | Search engine without medium |
| `bing` | Search Engine | `organic_search` | `organic` | Search engine without medium |
| `unknown_source` | No Fallback | `unassigned` | `unassigned` | Unrecognized source |

### **UTM Validation & Quality Control**

The system includes comprehensive UTM parameter validation to ensure data quality:

| **Validation Type** | **Examples** | **Result** | **Reason** |
|---------------------|--------------|------------|------------|
| **Source-Media Conflicts** | `google` + `paid_social` | `medium: 'unknown'` | Search engine with social medium |
| **Source-Media Conflicts** | `facebook` + `paid_search` | `medium: 'unknown'` | Social platform with search medium |
| **Source-Media Conflicts** | `youtube` + `email` | `medium: 'unknown'` | Video platform with email medium |
| **Invalid Placeholders** | `null`, `undefined`, `none` | `medium: 'unknown'` | Invalid parameter values |
| **Valid Combinations** | `google` + `cpc` | Normal classification | Legitimate search advertising |
| **Valid Combinations** | `facebook` + `display` | Normal classification | Legitimate social display ads |

#### **Validation Logic:**
- **Exact-Match Validation** - All normalized medium values are validated before classification
- **Comprehensive Coverage** - No UTM combination bypasses validation
- **Quality Flags** - `missing_medium: true` indicates validation failure
- **Debug Logging** - Failed validations are logged for monitoring
- **Unknown Fallback** - Invalid UTM combinations set `medium: 'unknown'` for external QA systems

### **Brand vs. Generic Detection**

| **UTM Campaign** | **Medium Override** | **Reason** |
|------------------|---------------------|------------|
| `brand-winter2024` | `paid-brand` | Contains "brand" |
| `branded-campaign` | `paid-brand` | Contains "branded" |
| `generic-shoes` | `paid-generic` | Contains "generic" |
| `winter-sale` | `cpc` | Standard CPC |

## 🎯 Summary: How Conflicts Are Resolved

### **1. Click ID Always Has Priority**
- **Channel Group** is determined by Click ID
- **UTM Parameters** are only accepted if plausible

### **2. Plausibility Check**
- **Source & Medium:** High error probability → Plausibility check required
- **Campaign, Term, Content:** Low error probability → Always accepted

### **3. Conflict Transparency**
- **Conflicts are flagged** but not hidden
- **Debugging information** for better data quality
- **Marketing teams** can correct UTM parameters

### **4. Fallback Strategy**
- **Without Click ID:** UTM parameters determine Channel Group
- **Without UTM:** Referrer analysis for Organic Traffic
- **Without Referrer:** Direct Traffic as fallback

**The system ensures precise attribution with maximum marketing flexibility!** 🎯

## 📊 Traffic Source Classification Matrix

### **🎯 PRIORITY 1: Click IDs (Highest Priority)**

| **Szenario** | **URL Parameter** | **Source** | **Medium** | **Channel Group** | **Ad Platform** | **Method** |
|--------------|-------------------|------------|------------|-------------------|-----------------|------------|
| **Google Ads Standard** | `?gclid=123` | `google` | `cpc` | `paid_search` | `google_ads` | `click_id` |
| **Google Ads Enhanced** | `?gbraid=456` | `google` | `cpc` | `paid_search` | `google_ads` | `click_id` |
| **Google Ads Enhanced** | `?wbraid=789` | `google` | `cpc` | `paid_search` | `google_ads` | `click_id` |
| **Microsoft Ads** | `?msclkid=abc` | `microsoft` | `cpc` | `paid_search` | `microsoft_ads` | `click_id` |
| **Facebook Ads** | `?fbclid=def` | `facebook` | `cpc` | `paid_social` | `facebook_ads` | `click_id` |
| **TikTok Ads** | `?ttclid=ghi` | `tiktok` | `cpc` | `paid_social` | `tiktok_ads` | `click_id` |
| **LinkedIn Ads** | `?li_fat_id=jkl` | `linkedin` | `cpc` | `paid_social` | `linkedin_ads` | `click_id` |
| **Twitter Ads** | `?twclid=mno` | `twitter` | `cpc` | `paid_social` | `twitter_ads` | `click_id` |

### **🎯 PRIORITY 2: UTM Parameter (Marketing-kontrolliert)**

#### **`paid_search` (CPC/PPC)**
| **Szenario** | **UTM Parameter** | **Source** | **Medium** | **Channel Group** | **Ad Platform** | **Method** |
|--------------|-------------------|------------|------------|-------------------|-----------------|------------|
| **Google Ads UTM** | `utm_source=google&utm_medium=cpc` | `google` | `cpc` | `paid_search` | - | `utm` |
| **Bing Ads UTM** | `utm_source=bing&utm_medium=ppc` | `bing` | `ppc` | `paid_search` | - | `utm` |
| **Brand Campaign** | `utm_source=google&utm_medium=cpc&utm_campaign=brand` | `google` | `paid-brand` | `paid_search` | - | `utm` |
| **Generic Campaign** | `utm_source=google&utm_medium=cpc&utm_campaign=generic` | `google` | `paid-generic` | `paid_search` | - | `utm` |

#### **`paid_social`**
| **Szenario** | **UTM Parameter** | **Source** | **Medium** | **Channel Group** | **Ad Platform** | **Method** |
|--------------|-------------------|------------|------------|-------------------|-----------------|------------|
| **Facebook Ads UTM** | `utm_source=facebook&utm_medium=cpc` | `facebook` | `cpc` | `paid_social` | - | `utm` |
| **Instagram Ads UTM** | `utm_source=instagram&utm_medium=cpc` | `instagram` | `cpc` | `paid_social` | - | `utm` |
| **LinkedIn Ads UTM** | `utm_source=linkedin&utm_medium=cpc` | `linkedin` | `cpc` | `paid_social` | - | `utm` |
| **TikTok Ads UTM** | `utm_source=tiktok&utm_medium=cpc` | `tiktok` | `cpc` | `paid_social` | - | `utm` |

#### **`paid_shopping`**
| **Szenario** | **UTM Parameter** | **Source** | **Medium** | **Channel Group** | **Ad Platform** | **Method** |
|--------------|-------------------|------------|------------|-------------------|-----------------|------------|
| **Amazon Ads** | `utm_source=amazon&utm_medium=cpc` | `amazon` | `cpc` | `paid_shopping` | - | `utm` |
| **Google Shopping** | `utm_source=google&utm_medium=cpc&utm_campaign=shopping` | `google` | `cpc` | `paid_shopping` | - | `utm` |
| **Shopify Ads** | `utm_source=shopify&utm_medium=cpc` | `shopify` | `cpc` | `paid_shopping` | - | `utm` |

#### **Display/Remarketing**
| **Szenario** | **UTM Parameter** | **Source** | **Medium** | **Channel Group** | **Ad Platform** | **Method** |
|--------------|-------------------|------------|------------|-------------------|-----------------|------------|
| **Display Banner** | `utm_source=criteo&utm_medium=display` | `criteo` | `display` | `display` | - | `utm` |
| **Remarketing** | `utm_source=google&utm_medium=remarketing&utm_campaign=retargeting` | `google` | `remarketing` | `display_remarketing` | - | `utm` |
| **Banner Ads** | `utm_source=adnxs&utm_medium=banner` | `adnxs` | `banner` | `display` | - | `utm` |

#### **Email Marketing**
| **Szenario** | **UTM Parameter** | **Source** | **Medium** | **Channel Group** | **Ad Platform** | **Method** |
|--------------|-------------------|------------|------------|-------------------|-----------------|------------|
| **Newsletter** | `utm_source=newsletter&utm_medium=email` | `newsletter` | `email` | `email` | - | `utm` |
| **Campaign Email** | `utm_source=campaign&utm_medium=e-mail` | `campaign` | `e-mail` | `email` | - | `utm` |
| **Transactional** | `utm_source=transactional&utm_medium=e_mail` | `transactional` | `e_mail` | `email` | - | `utm` |

#### **Affiliate Marketing**
| **Szenario** | **UTM Parameter** | **Source** | **Medium** | **Channel Group** | **Ad Platform** | **Method** |
|--------------|-------------------|------------|------------|-------------------|-----------------|------------|
| **Affiliate Partner** | `utm_source=partner&utm_medium=affiliate` | `partner` | `affiliate` | `affiliate` | - | `utm` |

#### **Mobile Push**
| **Szenario** | **UTM Parameter** | **Source** | **Medium** | **Channel Group** | **Ad Platform** | **Method** |
|--------------|-------------------|------------|------------|-------------------|-----------------|------------|
| **Push Notification** | `utm_source=app&utm_medium=push` | `app` | `push` | `mobile_push` | - | `utm` |

### **🔍 PRIORITY 3: `organic_search`**

| **Szenario** | **Referrer** | **Source** | **Medium** | **Channel Group** | **Search Term** | **Method** |
|--------------|--------------|------------|------------|-------------------|-----------------|------------|
| **Google Search** | `google.com/search?q=shoes` | `google` | `organic` | `organic_search` | `shoes` | `organic` |
| **Bing Search** | `bing.com/search?q=shoes` | `bing` | `organic` | `organic_search` | `shoes` | `organic` |
| **Yahoo Search** | `yahoo.com/search?p=shoes` | `yahoo` | `organic` | `organic_search` | `shoes` | `organic` |
| **DuckDuckGo** | `duckduckgo.com/?q=shoes` | `duckduckgo` | `organic` | `organic_search` | `shoes` | `organic` |
| **Yandex Search** | `yandex.com/search?text=shoes` | `yandex` | `organic` | `organic_search` | `shoes` | `organic` |
| **Baidu Search** | `baidu.com/s?wd=shoes` | `baidu` | `organic` | `organic_search` | `shoes` | `organic` |

#### **Organic Shopping**
| **Szenario** | **Referrer** | **Source** | **Medium** | **Channel Group** | **Search Term** | **Method** |
|--------------|--------------|------------|------------|-------------------|-----------------|------------|
| **Amazon Search** | `amazon.com/s?k=shoes` | `amazon` | `organic` | `organic_shopping` | - | `organic` |
| **Google Shopping** | `google.com/shopping` | `google` | `organic` | `organic_shopping` | - | `organic` |

#### **Organic Video**
| **Szenario** | **Referrer** | **Source** | **Medium** | **Channel Group** | **Search Term** | **Method** |
|--------------|--------------|------------|------------|-------------------|-----------------|------------|
| **YouTube Search** | `youtube.com/results?search_query=shoes` | `youtube` | `organic` | `organic_video` | - | `organic` |
| **Vimeo Search** | `vimeo.com/search?q=shoes` | `vimeo` | `organic` | `organic_video` | - | `organic` |

### **📱 PRIORITY 4: Social Media**

| **Szenario** | **Referrer** | **Source** | **Medium** | **Channel Group** | **Method** |
|--------------|--------------|------------|------------|-------------------|------------|
| **Facebook** | `facebook.com` | `facebook` | `social` | `organic_social` | `social` |
| **Instagram** | `instagram.com` | `instagram` | `social` | `organic_social` | `social` |
| **Twitter** | `twitter.com` | `twitter` | `social` | `organic_social` | `social` |
| **LinkedIn** | `linkedin.com` | `linkedin` | `social` | `organic_social` | `social` |
| **TikTok** | `tiktok.com` | `tiktok` | `social` | `organic_social` | `social` |
| **Pinterest** | `pinterest.com` | `pinterest` | `social` | `organic_social` | `social` |
| **YouTube** | `youtube.com` | `youtube` | `social` | `organic_video` | `social` |
| **Reddit** | `reddit.com` | `reddit` | `social` | `organic_social` | `social` |

### **📧 PRIORITY 5: Mail Clients**

| **Szenario** | **Referrer** | **Source** | **Medium** | **Channel Group** | **Method** |
|--------------|--------------|------------|------------|-------------------|------------|
| **Gmail** | `mail.google.com` | `mail` | `organic` | `email` | `mail` |
| **Outlook** | `outlook.live.com` | `mail` | `organic` | `email` | `mail` |
| **Yahoo Mail** | `mail.yahoo.com` | `mail` | `organic` | `email` | `mail` |
| **T-Online** | `t-online.de` | `mail` | `organic` | `email` | `mail` |

### **🔗 PRIORITY 6: Referral Traffic**

| **Szenario** | **Referrer** | **Source** | **Medium** | **Channel Group** | **Method** |
|--------------|--------------|------------|------------|-------------------|------------|
| **External Website** | `example.com` | `example.com` | `referral` | `referral` | `referral` |
| **Partner Site** | `partner.com` | `partner.com` | `referral` | `referral` | `referral` |
| **News Site** | `news.com` | `news.com` | `referral` | `referral` | `referral` |

### **🎯 PRIORITY 7: Direct Traffic (Fallback)**

| **Szenario** | **Referrer** | **Source** | **Medium** | **Channel Group** | **Method** |
|--------------|--------------|------------|------------|-------------------|------------|
| **Direct Visit** | `(none)` | `(direct)` | `(none)` | `direct` | `direct` |
| **Bookmark** | `(none)` | `(direct)` | `(none)` | `direct` | `direct` |
| **Typed URL** | `(none)` | `(direct)` | `(none)` | `direct` | `direct` |


## 📋 Conclusion

The GTM Traffic Source classification system represents a **strategic balance** between technical accuracy and marketing flexibility. By prioritizing **technical signals** while allowing **marketing control** through plausibility checks, the system ensures **reliable attribution** without sacrificing **campaign precision**.

This approach provides **competitive advantage** through superior data quality while maintaining **operational efficiency** through automated classification logic.

**Key Success Factors:**
1. **Technical Foundation** - Reliable Click ID detection
2. **Marketing Integration** - Flexible UTM parameter handling
3. **Quality Assurance** - Plausibility checks and validation
4. **Continuous Improvement** - Regular testing and optimization

The system is designed to **evolve with business needs** while maintaining **data integrity** and **marketing effectiveness**.

---

## License

MIT License - see [LICENSE](LICENSE) file for details.


## Support

For technical support and questions:
- **GitHub Issues** - Report bugs and request features
- **Documentation** - Check README and strategy documents
- **Code Examples** - See integration examples above

## Author

**/ MEDIAFAKTUR** – Marketing Performance Precision
[https://mediafaktur.marketing](https://mediafaktur.marketing)<br>
Florian Pankarter, [fp@mediafaktur.marketing](mailto:fp@mediafaktur.marketing)

---

**GTM Traffic Source Strategy** - Comprehensive decision logic and classification rules for traffic source attribution.
