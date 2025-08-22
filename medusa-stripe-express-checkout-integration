# medusa-stripe-express-checkout for nextjs starter template

## Summary

- **Uses CardElement** for checkout payment flow
- **Uses ExpressCheckoutElement** only in cart page
- **Single location Express Checkout** (cart page only, not multiple locations)
- **Simple payment session management** (one session per flow)
- **Minimal state management** with clean error handling
- **Direct payment confirmation** using confirmCardPayment for regular checkout
- **Proper order placement** with redirect to confirmation page
- **Express Checkout is isolated** - doesn't interfere with main checkout flow

# thanks lambdacurry for giving the integration in remix i just use next js starter here 

# My Working Stripe Integration Code

## File Paths and Code

### 1. Payment Component
**Path:** `storefornt/src/modules/checkout/components/payment/index.tsx`

```typescript
"use client"

import { RadioGroup } from "@headlessui/react"
import { isStripe as isStripeFunc, paymentInfoMap } from "@lib/constants"
import { initiatePaymentSession } from "@lib/data/cart"
import { CheckCircleSolid, CreditCard } from "@medusajs/icons"
import { Button, Container, Heading, Text, clx } from "@medusajs/ui"
import ErrorMessage from "@modules/checkout/components/error-message"
import PaymentContainer, {
  StripeCardContainer,
} from "@modules/checkout/components/payment-container"
import Divider from "@modules/common/components/divider"
import { usePathname, useRouter, useSearchParams } from "next/navigation"
import { useCallback, useEffect, useState } from "react"

const Payment = ({
  cart,
  availablePaymentMethods,
}: {
  cart: any
  availablePaymentMethods: any[]
}) => {
  const activeSession = cart.payment_collection?.payment_sessions?.find(
    (paymentSession: any) => paymentSession.status === "pending"
  )

  const [isLoading, setIsLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const [cardBrand, setCardBrand] = useState<string | null>(null)
  const [cardComplete, setCardComplete] = useState(false)
  const [selectedPaymentMethod, setSelectedPaymentMethod] = useState(
    activeSession?.provider_id ?? ""
  )

  const searchParams = useSearchParams()
  const router = useRouter()
  const pathname = usePathname()

  const isOpen = searchParams.get("step") === "payment"

  const isStripe = isStripeFunc(selectedPaymentMethod)

  const setPaymentMethod = async (method: string) => {
    setError(null)
    setSelectedPaymentMethod(method)
    if (isStripeFunc(method)) {
      await initiatePaymentSession(cart, {
        provider_id: method,
      })
    }
  }

  const paidByGiftcard =
    cart?.gift_cards && cart?.gift_cards?.length > 0 && cart?.total === 0

  const paymentReady =
    (activeSession && cart?.shipping_methods.length !== 0) || paidByGiftcard

  const createQueryString = useCallback(
    (name: string, value: string) => {
      const params = new URLSearchParams(searchParams)
      params.set(name, value)

      return params.toString()
    },
    [searchParams]
  )

  const handleEdit = () => {
    router.push(pathname + "?" + createQueryString("step", "payment"), {
      scroll: false,
    })
  }

  const handleSubmit = async () => {
    setIsLoading(true)
    try {
      const shouldInputCard =
        isStripeFunc(selectedPaymentMethod) && !activeSession

      const checkActiveSession =
        activeSession?.provider_id === selectedPaymentMethod

      if (!checkActiveSession) {
        await initiatePaymentSession(cart, {
          provider_id: selectedPaymentMethod,
        })
      }

      if (!shouldInputCard) {
        return router.push(
          pathname + "?" + createQueryString("step", "review"),
          {
            scroll: false,
          }
        )
      }
    } catch (err: any) {
      setError(err.message)
    } finally {
      setIsLoading(false)
    }
  }

  useEffect(() => {
    setError(null)
  }, [isOpen])

  return (
    <div className="bg-white">
      <div className="flex flex-row items-center justify-between mb-6">
        <Heading
          level="h2"
          className={clx(
            "flex flex-row text-3xl-regular gap-x-2 items-baseline",
            {
              "opacity-50 pointer-events-none select-none":
                !isOpen && !paymentReady,
            }
          )}
        >
          Payment
          {!isOpen && paymentReady && <CheckCircleSolid />}
        </Heading>
        {!isOpen && paymentReady && (
          <Text>
            <button
              onClick={handleEdit}
              className="text-ui-fg-interactive hover:text-ui-fg-interactive-hover"
              data-testid="edit-payment-button"
            >
              Edit
            </button>
          </Text>
        )}
      </div>
      <div>
        <div className={isOpen ? "block" : "hidden"}>
          {!paidByGiftcard && availablePaymentMethods?.length && (
            <>
              <RadioGroup
                value={selectedPaymentMethod}
                onChange={(value: string) => setPaymentMethod(value)}
              >
                {availablePaymentMethods.map((paymentMethod) => (
                  <div key={paymentMethod.id}>
                    {isStripeFunc(paymentMethod.id) ? (
                      <StripeCardContainer
                        paymentProviderId={paymentMethod.id}
                        selectedPaymentOptionId={selectedPaymentMethod}
                        paymentInfoMap={paymentInfoMap}
                        setCardBrand={setCardBrand}
                        setError={setError}
                        setCardComplete={setCardComplete}
                      />
                    ) : (
                      <PaymentContainer
                        paymentInfoMap={paymentInfoMap}
                        paymentProviderId={paymentMethod.id}
                        selectedPaymentOptionId={selectedPaymentMethod}
                      />
                    )}
                  </div>
                ))}
              </RadioGroup>
            </>
          )}

          {paidByGiftcard && (
            <div className="flex flex-col w-1/3">
              <Text className="txt-medium-plus text-ui-fg-base mb-1">
                Payment method
              </Text>
              <Text
                className="txt-medium text-ui-fg-subtle"
                data-testid="payment-method-summary"
              >
                Gift card
              </Text>
            </div>
          )}

          <ErrorMessage
            error={error}
            data-testid="payment-method-error-message"
          />

          <Button
            size="large"
            className="mt-6"
            onClick={handleSubmit}
            isLoading={isLoading}
            disabled={
              (isStripe && !cardComplete) ||
              (!selectedPaymentMethod && !paidByGiftcard)
            }
            data-testid="submit-payment-button"
          >
            {!activeSession && isStripeFunc(selectedPaymentMethod)
              ? " Enter card details"
              : "Continue to review"}
          </Button>
        </div>

        <div className={isOpen ? "hidden" : "block"}>
          {cart && paymentReady && activeSession ? (
            <div className="flex items-start gap-x-1 w-full">
              <div className="flex flex-col w-1/3">
                <Text className="txt-medium-plus text-ui-fg-base mb-1">
                  Payment method
                </Text>
                <Text
                  className="txt-medium text-ui-fg-subtle"
                  data-testid="payment-method-summary"
                >
                  {paymentInfoMap[activeSession?.provider_id]?.title ||
                    activeSession?.provider_id}
                </Text>
              </div>
              <div className="flex flex-col w-1/3">
                <Text className="txt-medium-plus text-ui-fg-base mb-1">
                  Payment details
                </Text>
                <div
                  className="flex gap-2 txt-medium text-ui-fg-subtle items-center"
                  data-testid="payment-details-summary"
                >
                  <Container className="flex items-center h-7 w-fit p-2 bg-ui-button-neutral-hover">
                    {paymentInfoMap[selectedPaymentMethod]?.icon || (
                      <CreditCard />
                    )}
                  </Container>
                  <Text>
                    {isStripeFunc(selectedPaymentMethod) && cardBrand
                      ? cardBrand
                      : "Another step will appear"}
                  </Text>
                </div>
              </div>
            </div>
          ) : paidByGiftcard ? (
            <div className="flex flex-col w-1/3">
              <Text className="txt-medium-plus text-ui-fg-base mb-1">
                Payment method
              </Text>
              <Text
                className="txt-medium text-ui-fg-subtle"
                data-testid="payment-method-summary"
              >
                Gift card
              </Text>
            </div>
          ) : null}
        </div>
      </div>
      <Divider className="mt-8" />
    </div>
  )
}

export default Payment
```

### 2. Payment Button Component
**Path:** `storefornt/src/modules/checkout/components/payment-button/index.tsx`

```typescript
"use client"

import { isManual, isStripe } from "@lib/constants"
import { placeOrder } from "@lib/data/cart"
import { HttpTypes } from "@medusajs/types"
import { Button } from "@medusajs/ui"
import { useElements, useStripe } from "@stripe/react-stripe-js"
import React, { useState } from "react"
import ErrorMessage from "../error-message"

type PaymentButtonProps = {
  cart: HttpTypes.StoreCart
  "data-testid": string
}

const PaymentButton: React.FC<PaymentButtonProps> = ({
  cart,
  "data-testid": dataTestId,
}) => {
  const notReady =
    !cart ||
    !cart.shipping_address ||
    !cart.billing_address ||
    !cart.email ||
    (cart.shipping_methods?.length ?? 0) < 1

  const paymentSession = cart.payment_collection?.payment_sessions?.[0]

  switch (true) {
    case isStripe(paymentSession?.provider_id):
      return (
        <StripePaymentButton
          notReady={notReady}
          cart={cart}
          data-testid={dataTestId}
        />
      )
    case isManual(paymentSession?.provider_id):
      return (
        <ManualTestPaymentButton notReady={notReady} data-testid={dataTestId} />
      )
    default:
      return <Button disabled>Select a payment method</Button>
  }
}

const StripePaymentButton = ({
  cart,
  notReady,
  "data-testid": dataTestId,
}: {
  cart: HttpTypes.StoreCart
  notReady: boolean
  "data-testid"?: string
}) => {
  const [submitting, setSubmitting] = useState(false)
  const [errorMessage, setErrorMessage] = useState<string | null>(null)

  const onPaymentCompleted = async () => {
    await placeOrder()
      .catch((err) => {
        setErrorMessage(err.message)
      })
      .finally(() => {
        setSubmitting(false)
      })
  }

  const stripe = useStripe()
  const elements = useElements()
  const card = elements?.getElement("card")

  const session = cart.payment_collection?.payment_sessions?.find(
    (s) => s.status === "pending"
  )

  const disabled = !stripe || !elements ? true : false

  const handlePayment = async () => {
    setSubmitting(true)

    if (!stripe || !elements || !card || !cart) {
      setSubmitting(false)
      return
    }

    await stripe
      .confirmCardPayment(session?.data.client_secret as string, {
        payment_method: {
          card: card,
          billing_details: {
            name:
              cart.billing_address?.first_name +
              " " +
              cart.billing_address?.last_name,
            address: {
              city: cart.billing_address?.city ?? undefined,
              country: cart.billing_address?.country_code ?? undefined,
              line1: cart.billing_address?.address_1 ?? undefined,
              line2: cart.billing_address?.address_2 ?? undefined,
              postal_code: cart.billing_address?.postal_code ?? undefined,
              state: cart.billing_address?.province ?? undefined,
            },
            email: cart.email,
            phone: cart.billing_address?.phone ?? undefined,
          },
        },
      })
      .then(({ error, paymentIntent }) => {
        if (error) {
          const pi = error.payment_intent

          if (
            (pi && pi.status === "requires_capture") ||
            (pi && pi.status === "succeeded")
          ) {
            onPaymentCompleted()
          }

          setErrorMessage(error.message || null)
          return
        }

        if (
          (paymentIntent && paymentIntent.status === "requires_capture") ||
          paymentIntent.status === "succeeded"
        ) {
          return onPaymentCompleted()
        }

        return
      })
  }

  return (
    <>
      <Button
        disabled={disabled || notReady}
        onClick={handlePayment}
        size="large"
        isLoading={submitting}
        data-testid={dataTestId}
      >
        Place order
      </Button>
      <ErrorMessage
        error={errorMessage}
        data-testid="stripe-payment-error-message"
      />
    </>
  )
}

const ManualTestPaymentButton = ({ notReady }: { notReady: boolean }) => {
  const [submitting, setSubmitting] = useState(false)
  const [errorMessage, setErrorMessage] = useState<string | null>(null)

  const onPaymentCompleted = async () => {
    await placeOrder()
      .catch((err) => {
        setErrorMessage(err.message)
      })
      .finally(() => {
        setSubmitting(false)
      })
  }

  const handlePayment = () => {
    setSubmitting(true)

    onPaymentCompleted()
  }

  return (
    <>
      <Button
        disabled={notReady}
        isLoading={submitting}
        onClick={handlePayment}
        size="large"
        data-testid="submit-order-button"
      >
        Place order
      </Button>
      <ErrorMessage
        error={errorMessage}
        data-testid="manual-payment-error-message"
      />
    </>
  )
}

export default PaymentButton
```

### 3. Payment Container
**Path:** `storefornt/src/modules/checkout/components/payment-container/index.tsx`

```typescript
import { Radio as RadioGroupOption } from "@headlessui/react"
import { Text, clx } from "@medusajs/ui"
import React, { useContext, useMemo, type JSX } from "react"

import Radio from "@modules/common/components/radio"

import { isManual } from "@lib/constants"
import SkeletonCardDetails from "@modules/skeletons/components/skeleton-card-details"
import { CardElement } from "@stripe/react-stripe-js"
import { StripeCardElementOptions } from "@stripe/stripe-js"
import PaymentTest from "../payment-test"
import { StripeContext } from "../payment-wrapper/stripe-wrapper"

type PaymentContainerProps = {
  paymentProviderId: string
  selectedPaymentOptionId: string | null
  disabled?: boolean
  paymentInfoMap: Record<string, { title: string; icon: JSX.Element }>
  children?: React.ReactNode
}

const PaymentContainer: React.FC<PaymentContainerProps> = ({
  paymentProviderId,
  selectedPaymentOptionId,
  disabled = false,
  paymentInfoMap,
  children,
}) => {
  return (
    <>
      <RadioGroupOption
        value={paymentProviderId}
        disabled={disabled}
        className={clx(
          "flex flex-col gap-y-2 text-small-regular cursor-pointer py-4 border rounded-rounded px-8 mb-2 hover:shadow-borders-interactive-with-active",
          {
            "border-ui-border-interactive":
              selectedPaymentOptionId === paymentProviderId,
          }
        )}
      >
        <div className="flex items-center justify-between ">
          <div className="flex items-center gap-x-4">
            <Radio checked={selectedPaymentOptionId === paymentProviderId} />
            <Text className="text-base-regular">
              {paymentInfoMap[paymentProviderId]?.title || paymentProviderId}
            </Text>
            {paymentInfoMap[paymentProviderId]?.icon}
          </div>
        </div>
        {isManual(paymentProviderId) && (
          <PaymentTest className="hidden small:block" />
        )}
        {children}
      </RadioGroupOption>
    </>
  )
}

export default PaymentContainer

export const StripeCardContainer = ({
  paymentProviderId,
  selectedPaymentOptionId,
  paymentInfoMap,
  disabled = false,
  setCardBrand,
  setError,
  setCardComplete,
}: Omit<PaymentContainerProps, "children"> & {
  setCardBrand: (brand: string) => void
  setError: (error: string | null) => void
  setCardComplete: (complete: boolean) => void
}) => {
  const stripeReady = useContext(StripeContext)

  const useOptions: StripeCardElementOptions = useMemo(() => {
    return {
      style: {
        base: {
          fontFamily: "Inter, sans-serif",
          color: "#424270",
          "::placeholder": {
            color: "rgb(107 114 128)",
          },
        },
      },
      classes: {
        base: "pt-3 pb-1 block w-full h-11 px-4 mt-0 bg-ui-bg-field border rounded-md appearance-none focus:outline-none focus:ring-0 focus:shadow-borders-interactive-with-active border-ui-border-base hover:bg-ui-bg-field-hover transition-all duration-300 ease-in-out",
      },
    }
  }, [])

  return (
    <PaymentContainer
      paymentProviderId={paymentProviderId}
      selectedPaymentOptionId={selectedPaymentOptionId}
      paymentInfoMap={paymentInfoMap}
      disabled={disabled}
    >
      {selectedPaymentOptionId === paymentProviderId &&
        (stripeReady ? (
          <div className="my-4 transition-all duration-150 ease-in-out">
            <Text className="txt-medium-plus text-ui-fg-base mb-1">
              Enter your card details:
            </Text>
            <CardElement
              options={useOptions as StripeCardElementOptions}
              onChange={(e) => {
                setCardBrand(
                  e.brand && e.brand.charAt(0).toUpperCase() + e.brand.slice(1)
                )
                setError(e.error?.message || null)
                setCardComplete(e.complete)
              }}
            />
          </div>
        ) : (
          <SkeletonCardDetails />
        ))}
    </PaymentContainer>
  )
}
```

### 4. Stripe Wrapper
**Path:** `storefornt/src/modules/checkout/components/payment-wrapper/stripe-wrapper.tsx`

```typescript
"use client"

import { Stripe, StripeElementsOptions } from "@stripe/stripe-js"
import { Elements } from "@stripe/react-stripe-js"
import { HttpTypes } from "@medusajs/types"
import { createContext } from "react"

type StripeWrapperProps = {
  paymentSession: HttpTypes.StorePaymentSession
  stripeKey?: string
  stripePromise: Promise<Stripe | null> | null
  children: React.ReactNode
}

export const StripeContext = createContext(false)

const StripeWrapper: React.FC<StripeWrapperProps> = ({
  paymentSession,
  stripeKey,
  stripePromise,
  children,
}) => {
  const options: StripeElementsOptions = {
    clientSecret: paymentSession!.data?.client_secret as string | undefined,
  }

  if (!stripeKey) {
    throw new Error(
      "Stripe key is missing. Set NEXT_PUBLIC_STRIPE_KEY environment variable."
    )
  }

  if (!stripePromise) {
    throw new Error(
      "Stripe promise is missing. Make sure you have provided a valid Stripe key."
    )
  }

  if (!paymentSession?.data?.client_secret) {
    throw new Error(
      "Stripe client secret is missing. Cannot initialize Stripe."
    )
  }

  return (
    <StripeContext.Provider value={true}>
      <Elements options={options} stripe={stripePromise}>
        {children}
      </Elements>
    </StripeContext.Provider>
  )
}

export default StripeWrapper
```

### 5. PlaceOrder Function
**Path:** `storefornt/src/lib/data/cart.ts` (placeOrder function)

```typescript
export async function placeOrder(cartId?: string) {
  const id = cartId || (await getCartId())

  if (!id) {
    throw new Error("No existing cart found when placing an order")
  }

  const headers = {
    ...(await getAuthHeaders()),
  }

  const cartRes = await sdk.store.cart
    .complete(id, {}, headers)
    .then(async (cartRes) => {
      const cartCacheTag = await getCacheTag("carts")
      revalidateTag(cartCacheTag)
      return cartRes
    })
    .catch(medusaError)

  if (cartRes?.type === "order") {
    const countryCode =
      cartRes.order.shipping_address?.country_code?.toLowerCase()

    const orderCacheTag = await getCacheTag("orders")
    revalidateTag(orderCacheTag)

    removeCartId()
    redirect(`/${countryCode}/order/${cartRes?.order.id}/confirmed`)
  }

  return cartRes.cart
}
```

### 6. Express Checkout Component
**Path:** `storefornt/src/modules/cart/components/express-checkout /index.tsx`

```typescript
import { useState, useEffect } from "react"
import { useStripe, useElements, ExpressCheckoutElement } from "@stripe/react-stripe-js"
import { initiatePaymentSession, placeOrder } from "@lib/data/cart"
import { HttpTypes } from "@medusajs/types"

interface ExpressCheckoutProps {
  cart?: HttpTypes.StoreCart
  countryCode?: string
}

export default function ExpressCheckout({ cart, countryCode }: ExpressCheckoutProps) {
  const stripe = useStripe()
  const elements = useElements()
  const [isProcessing, setIsProcessing] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const [isElementReady, setIsElementReady] = useState(false)
  const [shippingOptions, setShippingOptions] = useState<ShippingOption[]>([])
  const [currentCart, setCurrentCart] = useState(cart)
  const [clientSecret, setClientSecret] = useState<string | null>(null)

  // Create payment intent with base cart amount - simple and safe approach
  useEffect(() => {
    const initializePayment = async () => {
      try {
        console.log("💳 Creating initial payment session with base cart:", currentCart.total)
        
        const paymentSessionResponse = await initiatePaymentSession(currentCart, {
          provider_id: "pp_stripe_stripe",
        })

        const paymentSession = paymentSessionResponse?.payment_collection?.payment_sessions?.[0]
        const initialClientSecret = paymentSession?.data?.client_secret as string

        if (initialClientSecret) {
          setClientSecret(initialClientSecret)
          console.log("✅ Initial payment session created with base amount:", currentCart.total)
        }
      } catch (error) {
        console.error("Failed to initialize payment session:", error)
        setError("Failed to initialize payment")
      }
    }

    if (currentCart?.id && !clientSecret) {
      initializePayment()
    }
  }, [currentCart?.id, clientSecret])

  const handleExpressCheckout = async (event: any) => {
    setIsProcessing(true)
    setError(null)

    try {
      console.log("💳 Express checkout confirmed with full event data:", event)

      // Extract customer information
      const customerEmail = event.billingDetails?.email || ""
      const customerName = event.billingDetails?.name || ""

      // Update cart with customer info and place order
      const updatedCart = await updateCart({
        email: customerEmail,
        billing_address: {
          first_name: customerName.split(" ")[0] || "",
          last_name: customerName.split(" ").slice(1).join(" ") || "",
          // ... other address fields
        }
      })

      // Place the order
      const orderResult = await placeOrder(updatedCart.id)
      
      if (!orderResult?.id) {
        throw new Error("Failed to place order")
      }

      console.log("✅ Order placed successfully:", orderResult.id)
      
      // Redirect to confirmation
      window.location.href = `/${countryCode}/order/${orderResult.id}/confirmed`
      
    } catch (error: any) {
      console.error("❌ Express checkout failed:", error)
      setError(error.message || "Express checkout failed")
    } finally {
      setIsProcessing(false)
    }
  }

  const handleShippingAddressChange = async (event: any) => {
    // Simple address handling - just resolve with basic shipping rate
    try {
      event.resolve({
        shippingRates: [{
          id: 'standard-shipping',
          displayName: 'Standard Shipping',
          amount: 743, // $7.43 in cents
          deliveryEstimate: {
            minimum: { unit: 'business_day', value: 3 },
            maximum: { unit: 'business_day', value: 7 }
          }
        }]
      })
    } catch (error) {
      console.error("Shipping address change error:", error)
      event.reject({ reason: "shipping_address_unserviceable" })
    }
  }

  const handleShippingRateChange = (event: any) => {
    // Simple rate change handling
    event.resolve()
  }

  if (!clientSecret || !isElementReady) {
    return (
      <div className="express-checkout-loading">
        <div className="animate-pulse bg-gray-200 h-12 rounded"></div>
      </div>
    )
  }

  return (
    <div className="express-checkout-container">
      <ExpressCheckoutElement
        onConfirm={handleExpressCheckout}
        onShippingAddressChange={handleShippingAddressChange}
        onShippingRateChange={handleShippingRateChange}
        options={{
          layout: {
            maxColumns: 1,
            maxRows: 1,
            overflow: 'auto',
          },
          buttonTheme: {
            applePay: "black",
            googlePay: "black",
            paypal: "gold",
          },
          buttonType: {
            applePay: "buy",
            googlePay: "buy", 
            paypal: "buynow",
          },
          buttonHeight: 48,
          shippingAddressRequired: true,
          emailRequired: true,
          phoneNumberRequired: true,
          shippingRates: [{
            id: 'estimated-shipping',
            displayName: 'Estimated Shipping',
            amount: 743, // $7.43 in cents
            deliveryEstimate: {
              minimum: { unit: 'business_day', value: 3 },
              maximum: { unit: 'business_day', value: 7 }
            }
          }],
        }}
        onReady={(event) => {
          console.log("✅ ExpressCheckoutElement ready")
          setIsElementReady(true)
        }}
        onLoadError={(event) => {
          console.error("❌ ExpressCheckoutElement load error", event)
          setError("Failed to load express checkout options")
        }}
      />

      {isElementReady && (
        <div className="flex items-center my-8">
          <div className="flex-1 border-t border-gray-300"></div>
          <div className="mx-4 text-gray-500 text-sm">or</div>
          <div className="flex-1 border-t border-gray-300"></div>
        </div>
      )}

      {error && (
        <div className="mt-4 p-3 bg-red-50 border border-red-200 rounded text-red-700 text-sm">
          {error}
        </div>
      )}
    </div>
  )
}
```

### 7. Express Checkout Wrapper
**Path:** `storefornt/src/modules/cart/components/express-checkout /wrapper.tsx`

```typescript
import { Elements } from "@stripe/react-stripe-js"
import { loadStripe } from "@stripe/stripe-js"
import ExpressCheckout from "./index"
import { HttpTypes } from "@medusajs/types"

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_KEY!)

interface ExpressCheckoutWrapperProps {
  cart?: HttpTypes.StoreCart
  countryCode?: string
}

export default function ExpressCheckoutWrapper({
  cart,
  countryCode,
}: ExpressCheckoutWrapperProps) {
  // Don't render if no cart or cart is empty
  if (!cart?.id || !cart.items?.length) {
    return null
  }

  // Check if Stripe is configured
  if (!process.env.NEXT_PUBLIC_STRIPE_KEY) {
    if (process.env.NODE_ENV === "development") {
      return (
        <div className="bg-yellow-50 p-3 rounded text-xs text-yellow-800 border border-yellow-200">
          ⚠️ <strong>Missing Stripe Configuration:</strong>
          <br />
          Add NEXT_PUBLIC_STRIPE_KEY to your .env.local file
        </div>
      )
    }
    return null
  }

  // Stripe Elements options for Express Checkout with shipping
  const options = {
    mode: "payment" as const,
    amount: Math.round((cart.total || 0) * 100), // Convert to cents
    currency: cart.currency_code?.toLowerCase() || "usd",
    appearance: {
      theme: "stripe" as const,
    },
  }

  return (
    <Elements stripe={stripePromise} options={options}>
      <ExpressCheckout cart={cart} countryCode={countryCode} />
    </Elements>
  )
}
```

### 8. Cart Template with Express Checkout
**Path:** `storefornt/src/modules/cart/templates/index.tsx`

```typescript
import ItemsTemplate from "./items"
import Summary from "./summary"
import EmptyCartMessage from "../components/empty-cart-message"
import SignInPrompt from "../components/sign-in-prompt"
import Divider from "@modules/common/components/divider"
import ExpressCheckoutWrapper from "../components/express-checkout /wrapper"
import { HttpTypes } from "@medusajs/types"

const CartTemplate = ({
  cart,
  customer,
  countryCode,
}: {
  cart: HttpTypes.StoreCart | null
  customer: HttpTypes.StoreCustomer | null
  countryCode?: string
}) => {
  return (
    <div className="py-12">
      <div className="content-container" data-testid="cart-container">
        {cart?.items?.length ? (
          <div className="grid grid-cols-1 small:grid-cols-[1fr_360px] gap-x-40">
            <div className="flex flex-col bg-white py-6 gap-y-6">
              {!customer && (
                <>
                  <SignInPrompt />
                  <Divider />
                </>
              )}
              <ItemsTemplate cart={cart} />
            </div>
            <div className="relative">
              <div className="flex flex-col gap-y-8 sticky top-12">
                {cart && cart.region && (
                  <>
                    {/* Express Checkout - Only show when cart has items */}
                    {cart.items && cart.items.length > 0 && (
                      <div className="bg-white py-6">
                        <ExpressCheckoutWrapper cart={cart as any} countryCode={countryCode} />
                      </div>
                    )}
                    
                    <div className="bg-white py-6">
                      <Summary cart={cart as any} />
                    </div>
                  </>
                )}
              </div>
            </div>
          </div>
        ) : (
          <div>
            <EmptyCartMessage />
          </div>
        )}
      </div>
    </div>
  )
}

export default CartTemplate
```

