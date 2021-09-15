# stripe-subscription

First of All  Create plan in strip account
Then past Stripe Credentials in .env file and plan_id
this is only for one plan subscription but if you have 2,3 or more plan then make a table in database fetch plan from it and pass this plan
where i am passing plan_id in each place

i only passing plan_id in a controller function you have to past plan id according to your desire
now past in .env file your account public key , secret and plan_id


STRIPE_KEY=pk_test_51IQ7cFAFpOTdBwUqzuZbmN15lsrGtucmPCEV66yBot9to7jKxt253szywUu demo is here(these are not correct)
STRIPE_SECRET=sk_test_51IQ7cFAFpOTdBwUqi5a4F19r7MbL8YfLZpCndIKX0VrqREEOdoQPPKBeW demo is here(these are not correct)
STRIPE_PLAN_PRICE=price_1I demo is here(these are not correct)




# Step1 install cachier 

composer require laravel/cashier

#step 2 run php artisan migrate after making changes in user table

 $table->string('stripe_id')->nullable();
    $table->string('card_brand')->nullable();
    $table->string('card_last_four')->nullable();
    $table->timestamp('trial_ends_at')->nullable();

# step 3 subscription.blade.php

add above subscription.blade.php file 

# step 4 past this method in your require controller

public function getStripeSubscription($request)
    {

        $user = auth()->user();
        $input = $request->all();
        $token =  $request->stripeToken;
        $paymentMethod = $request->paymentMethod;
        try {

            Stripe::setApiKey(env('STRIPE_SECRET'));

            if (is_null($user->stripe_id)) {
                $stripeCustomer = $user->createAsStripeCustomer();
            }

            \Stripe\Customer::createSource(
                $user->stripe_id,
                ['source' => $token]
            );

            $user->newSubscription('Premium Plan',env('STRIPE_PLAN_PRICE'))
                ->create($paymentMethod, [
                    'email' => $user->email,
                ]);

            return $user;
        } catch (Exception $e) {
            return back()->with('success',$e->getMessage());
        }

    }
    
    # step 5 route
    Route::get('subscription', ['as'=>'subscription','uses'=>'HomeController@subscription']);
    Route::post('subscription', ['as'=>'post-subscription','uses'=>'HomeController@getStripeSubscription']);


