# Amplitude Setup Guide

## Step 1: Define Events in Amplitude

Based on `DA.md`, the following events need to be defined in your Amplitude project.

- `landing_enter`
- `intro_step_complete`
- `enter_character_select`
- `character_select`
- `user_info_submit`
- `result_received`
- `payment_click`
- `payment_complete`
- `paid_version_enter`
- `paid_version_complete_read`
- `share_click`
- `unselected_character_shown`
- `unselected_character_click`

## Step 2: Implement Event Tracking

Here's how to implement tracking for each event, based on the `DA.md` documentation.

### 1. `landing_enter`
- **When:** Main page component mounts.
- **Where:** Main page component `useEffect`.
- **Purpose:** Track service entry and analyze inbound channels.
- **Code:**
  ```typescript
  useEffect(() => {
    const params = new URLSearchParams(window.location.search);
    trackEvent('landing_enter', {
      utm_source: params.get('utm_source'),
      utm_medium: params.get('utm_medium'),
      utm_campaign: params.get('utm_campaign'),
      referral_code: params.get('ref'),
    });
  }, []);
  ```

### 2. `intro_step_complete`
- **When:** User clicks to proceed to the next intro step.
- **Where:** Intro sequence button click handler.
- **Purpose:** Calculate step-by-step CTR and drop-off rates.
- **Code:**
  ```typescript
  const handleNextStep = (currentStep: number, totalSteps: number) => {
    trackEvent('intro_step_complete', {
      step_index: currentStep,
      step_total: totalSteps,
    });
    // ... proceed to next step
  };
  ```

### 3. `enter_character_select`
- **When:** Character selection screen component mounts.
- **Where:** Character selection page component `useEffect`.
- **Purpose:** Start measuring time spent on character selection.
- **Code:**
  ```typescript
  useEffect(() => {
    trackEvent('enter_character_select', {
      timestamp_enter: Date.now(),
    });
  }, []);
  ```

### 4. `character_select`
- **When:** User selects a character.
- **Where:** Character selection button click handler.
- **Purpose:** Finish measuring time spent on selection; analyze character-to-payment correlation.
- **Code:**
  ```typescript
  const handleCharacterSelect = (characterId: 'A' | 'B') => {
    trackEvent('character_select', {
      character_id: characterId,
      timestamp_select: Date.now(),
    });
    // ... handle selection
  };
  ```

### 5. `user_info_submit`
- **When:** User submits their information to generate results.
- **Where:** Information form submit handler.
- **Purpose:** Track user flow progression. **Do not include PII.**
- **Code:**
  ```typescript
  const handleSubmit = (characterId: 'A' | 'B') => {
    trackEvent('user_info_submit', {
      character_id: characterId,
    });
    // ... submit for results
  };
  ```

### 6. `result_received`
- **When:** Results page component mounts (after data loads).
- **Where:** Results page component `useEffect`.
- **Purpose:** Denominator for calculating paid conversion rate.
- **Code:**
  ```typescript
  useEffect(() => {
    if (resultLoaded) {
      trackEvent('result_received', {
        character_id: characterId,
        result_type: isPaid ? 'paid' : 'free',
      });
    }
  }, [resultLoaded]);
  ```

### 7. `payment_click`
- **When:** User clicks the payment button.
- **Where:** Payment button click handler.
- **Purpose:** Measure payment intent.
- **Code:**
  ```typescript
  const handlePaymentClick = (characterId: 'A' | 'B') => {
    trackEvent('payment_click', {
      character_id: characterId,
    });
    // ... enter payment flow
  };
  ```

### 8. `payment_complete`
- **When:** After receiving PayApp feedback URL callback (`state=1`).
- **Where:** Backend API route (`/api/payment/callback/route.ts`). **Server-side only.**
- **Purpose:** Numerator for paid conversion rate.
- **Code (Backend):**
  ```typescript
  // app/api/payment/callback/route.ts
  if (state === '1') {
    await fetch('https://api2.amplitude.com/2/httpapi', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        api_key: process.env.AMPLITUDE_API_KEY,
        events: [{
          event_type: 'payment_complete',
          user_id: internalUserId,
          event_properties: {
            character_id: characterId,
            payment_method: 'payapp',
            mul_no: mulNo,
          },
        }],
      }),
    });
  }
  ```

### 9. `paid_version_enter`
- **When:** Paid result page component mounts.
- **Where:** Paid result page `useEffect`.
- **Purpose:** Start measuring time-to-read for paid content.
- **Code:**
  ```typescript
  useEffect(() => {
    trackEvent('paid_version_enter', {
      character_id: characterId,
      timestamp_enter: Date.now(),
    });
  }, []);
  ```

### 10. `paid_version_complete_read`
- **When:** User scrolls to the bottom of the content.
- **Where:** Scroll detection logic (e.g., IntersectionObserver).
- **Purpose:** Finish measuring time-to-read.
- **Code:**
  ```typescript
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          trackEvent('paid_version_complete_read', {
            character_id: characterId,
            timestamp_complete: Date.now(),
          });
          observer.disconnect();
        }
      },
      { threshold: 1.0 }
    );
    // ... observe bottom element
  }, []);
  ```

### 11. `share_click`
- **When:** User clicks the share button.
- **Where:** Share button click handler.
- **Purpose:** Measure share CTR and viral loop.
- **Code:**
  ```typescript
  const handleShareClick = (characterId: 'A' | 'B', referralCode: string) => {
    trackEvent('share_click', {
      character_id: characterId,
      referral_code: referralCode,
    });
    // ... execute share action
  };
  ```

### 12. `unselected_character_shown`
- **When:** The UI for the unselected character enters the viewport.
- **Where:** IntersectionObserver for the unselected character component.
- **Purpose:** Denominator for calculating click-through on unselected character.
- **Code:**
  ```typescript
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          trackEvent('unselected_character_shown', {
            shown_character_id: unselectedCharacterId,
          });
          observer.disconnect();
        }
      },
      { threshold: 0.5 }
    );
    // ... observe element
  }, []);
  ```

### 13. `unselected_character_click`
- **When:** User clicks on the unselected character.
- **Where:** Click handler for the unselected character button.
- **Purpose:** Numerator for calculating click-through on unselected character.
- **Code:**
  ```typescript
  const handleUnselectedCharacterClick = (unselectedCharacterId: 'A' | 'B') => {
    trackEvent('unselected_character_click', {
      shown_character_id: unselectedCharacterId,
    });
    // ... enter flow for that character
  };
  ```

## Step 3: Set Up Dashboards and Funnels

This section, from `hmda.md`, outlines the "Why" behind the event tracking. Use this to build your Amplitude charts.

### [HMDA-1] Activation Measurement
- **Goal:** Measure CTR and drop-off rates for each step of the initial user flow.
- **Events:** `landing_enter`, `intro_step_complete`, `enter_character_select`, `character_select`
- **Amplitude Chart:** Funnel chart to visualize the flow and identify where users drop off.

### [HMDA-2] Transition Measurement
- **Goal:** Understand the user journey from landing to result, segmented by acquisition channel.
- **Events:** `landing_enter`, `character_select`, `result_received`
- **Properties:** `utm_source`, `utm_medium`, `utm_campaign`
- **Amplitude Chart:** User Path or Journeys chart. Use Funnel charts filtered by UTM properties to compare channels.

### [HMDA-3] Retention / System Measurement
- **Goal:** Measure paid conversion, sharing, and engagement with other characters.
- **Events:** `result_received`, `payment_click`, `payment_complete`, `share_click`, `unselected_character_shown`, `unselected_character_click`
- **Amplitude Chart:**
    - **Funnel:** `result_received` -> `payment_complete` for conversion rate.
    - **Segmentation:** Analyze `payment_complete` events grouped by `character_id`.
    - **Metrics:** Ratios of `share_click` to `result_received`, and `unselected_character_click` to `unselected_character_shown`.

### [HMDA-4] Time Measurement
- **Goal:** Measure how long it takes users to read the paid content.
- **Events:** `paid_version_enter`, `paid_version_complete_read`
- **Amplitude Chart:** Calculate the interval between the two events. A histogram is useful to see the distribution of time spent.

### [HMDA-5] Loop Measurement
- **Goal:** Measure the effectiveness of the viral loop (sharing).
- **Events:** `share_click`, `landing_enter` (with `referral_code` property)
- **Amplitude Chart:** Funnel from `landing_enter` to `share_click`. Track new `landing_enter` events that have a `referral_code`.

### [HMDA-6] UTM Naming Convention
- **Goal:** Ensure consistent and clean data from marketing campaigns.
- **Action:** Define and enforce a strict naming convention for `utm_source`, `utm_medium`, and `utm_campaign`. For example, use lowercase and underscores instead of spaces. This is a data governance task, not an implementation one, but it's crucial for useful analytics.
