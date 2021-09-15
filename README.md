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

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Monthly Subscription App using Stripe, Cashier and Laravel 5.4 with example</title>
    <!-- Styles -->
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="http://formvalidation.io/vendor/formvalidation/css/formValidation.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    <script src="http://formvalidation.io/vendor/formvalidation/js/formValidation.min.js"></script>
    <script src="http://formvalidation.io/vendor/formvalidation/js/framework/bootstrap.min.js"></script>
</head>
<body>
<div class="row">
<form id="paymentForm" class="form-horizontal">
    <input type="hidden" name="_token" value="{{csrf_token()}}">
    <div class="form-group">
        <label class="col-xs-3 control-label">Subscription Plan</label>
        <div class="col-xs-5">
            <select name="subscription" class="form-control">
                <option value="diamond">Diamond ($20.00/month)</option>
            </select>
        </div>
    </div>
    <div class="form-group">
        <label class="col-xs-3 control-label">Name</label>
        <div class="col-xs-5">
            <input type="text" class="form-control" name="name" />
        </div>
    </div>
    <div class="form-group">
        <label class="col-xs-3 control-label">Email</label>
        <div class="col-xs-5">
            <input type="email" class="form-control" name="email" />
        </div>
    </div>
    <div class="form-group">
        <label class="col-xs-3 control-label">Password</label>
        <div class="col-xs-5">
            <input type="password" class="form-control" name="password" />
        </div>
    </div>
    <div class="form-group">
        <label class="col-xs-3 control-label">Credit card number</label>
        <div class="col-xs-5">
            <input type="text" class="form-control" data-stripe="number" />
        </div>
    </div>
    <div class="form-group">
        <label class="col-xs-3 control-label">Expiration</label>
        <div class="col-xs-3">
            <input type="text" class="form-control" placeholder="Month" data-stripe="exp-month" />
        </div>
        <div class="col-xs-2">
            <input type="text" class="form-control" placeholder="Year" data-stripe="exp-year" />
        </div>
    </div>
    <div class="form-group">
        <label class="col-xs-3 control-label">CVV</label>
        <div class="col-xs-2">
            <input type="text" class="form-control" data-stripe="cvc" />
        </div>
    </div>
    <div class="form-group">
        <div class="col-xs-9 col-xs-offset-3">
            <button type="submit" class="btn btn-primary">Sign Up</button>
        </div>
    </div>
    <input type="hidden" name="token" value="" />
</form>
</div>
<script src="https://js.stripe.com/v2/"></script>
<script>
$(document).ready(function() {
    // Change the key to your one
    Stripe.setPublishableKey('your_stripe_key');
    $('#paymentForm')
        .formValidation({
            framework: 'bootstrap',
            icon: {
                valid: 'glyphicon glyphicon-ok',
                invalid: 'glyphicon glyphicon-remove',
                validating: 'glyphicon glyphicon-refresh'
            },
            fields: {
                name: {
                    validators: {
                        notEmpty: {
                            message: 'The name is required'
                        }
                    }
                },
                email: {
                    validators: {
                        notEmpty: {
                            message: 'The email is required'
                        }
                    }
                },
                password: {
                    validators: {
                        notEmpty: {
                            message: 'The password is required'
                        }
                    }
                },
                ccNumber: {
                    selector: '[data-stripe="number"]',
                    validators: {
                        notEmpty: {
                            message: 'The credit card number is required'
                        },
                        creditCard: {
                            message: 'The credit card number is not valid'
                        }
                    }
                },
                expMonth: {
                    selector: '[data-stripe="exp-month"]',
                    row: '.col-xs-3',
                    validators: {
                        notEmpty: {
                            message: 'The expiration month is required'
                        },
                        digits: {
                            message: 'The expiration month can contain digits only'
                        },
                        callback: {
                            message: 'Expired',
                            callback: function(value, validator) {
                                value = parseInt(value, 10);
                                var year         = validator.getFieldElements('expYear').val(),
                                    currentMonth = new Date().getMonth() + 1,
                                    currentYear  = new Date().getFullYear();
                                if (value < 0 || value > 12) {
                                    return false;
                                }
                                if (year == '') {
                                    return true;
                                }
                                year = parseInt(year, 10);
                                if (year > currentYear || (year == currentYear && value >= currentMonth)) {
                                    validator.updateStatus('expYear', 'VALID');
                                    return true;
                                } else {
                                    return false;
                                }
                            }
                        }
                    }
                },
                expYear: {
                    selector: '[data-stripe="exp-year"]',
                    row: '.col-xs-3',
                    validators: {
                        notEmpty: {
                            message: 'The expiration year is required'
                        },
                        digits: {
                            message: 'The expiration year can contain digits only'
                        },
                        callback: {
                            message: 'Expired',
                            callback: function(value, validator) {
                                value = parseInt(value, 10);
                                var month        = validator.getFieldElements('expMonth').val(),
                                    currentMonth = new Date().getMonth() + 1,
                                    currentYear  = new Date().getFullYear();
                                if (value < currentYear || value > currentYear + 100) {
                                    return false;
                                }
                                if (month == '') {
                                    return false;
                                }
                                month = parseInt(month, 10);
                                if (value > currentYear || (value == currentYear && month >= currentMonth)) {
                                    validator.updateStatus('expMonth', 'VALID');
                                    return true;
                                } else {
                                    return false;
                                }
                            }
                        }
                    }
                },
                cvvNumber: {
                    selector: '[data-stripe="cvc"]',
                    validators: {
                        notEmpty: {
                            message: 'The CVV number is required'
                        },
                        cvv: {
                            message: 'The value is not a valid CVV',
                            creditCardField: 'ccNumber'
                        }
                    }
                }
            }
        })
        .on('success.form.fv', function(e) {
            e.preventDefault();
            var $form = $(e.target);
            // Reset the token first
            $form.find('[name="token"]').val('');
            Stripe.card.createToken($form, function(status, response) {
                if (response.error) {
                    alert(response.error.message);
                } else {                  
                    // Set the token value
                    $form.find('[name="token"]').val(response.id);                 
                    // Or using Ajax
                    $.ajax({
                        // You need to change the url option to your back-end endpoint
                        url: "{{route('post-subscription')}}",
                        data: $form.serialize(),
                        method: 'POST',
                        dataType: 'json'
                    }).success(function(data) {
                        alert(data.msg);                        
                        // Reset the form
                        $form.formValidation('resetForm', true);
                    });
                }
            });
        });
});
</script>
</body>
</html>


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


