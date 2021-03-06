Component-Clinic
================

[![Travis Status](https://travis-ci.org/MyPost/component-clinic.svg)](https://travis-ci.org/MyPost/component-clinic)

[![Boo-Boo Bot by Jenn and Tony Bot, on Flickr
	](https://farm4.staticflickr.com/3641/3661635778_a7793d730c_q.jpg)
   ](https://www.flickr.com/photos/ittybittiesforyou/3661635778)

## PSA: Your components may be sick!

Make sure they are having regular checkups at the component-clinic!

* A small helper library to allow components to be made healthy.
* Useful for treating components that may become diseased on-the-fly.
* Initialize sickly components to facilitate crash-driven development.
* Works well with [Stuart Sierra components](https://github.com/stuartsierra/component),
  but doesn't depend on them.

## Usage

Leiningen: See [Clojars.](https://clojars.org/au.com.auspost/component-clinic)

In your code:

		(require '[component-clinic.core :as cc])

## Features

We provide two protocols (with one function each)

* curable      (treat!   this)
* diagnosable  (healthy? this)

and two additional functions

* (attend!    this options*)
* (discharge! this)

... respectively.

This library is intended to act on a patient (object), who at the bare minimum:

* Implements curable
* Is at some-point declared under care through the use of 'attend!

If you do this, then your patient will be checked periodically
(defaulting to 10 :seconds) to see if they are healthy (defaulting to sick)
and if found unwell, will be healed using your definition of 'treat!
in the implementation of the curable protocol.

If you wish to provide your own diagnostics, then you may also implement
the diagnosable protocol (with its 'healthy? method) in order to
test well-being. A falsy response will indicate that the patient is sick.

The interval of diagnosis repetition is provided as an option
to 'attend! as :checkup-interval. If this option is missing, then
the same key of your patient will be used.
Finally, if that key is missing too, then the interval will default to
10 seconds. Intervals are indicated either with milliseconds,
or with a tuple of [magnitude, units] (see 'get-time).

If you wish to discharge your patient, then you may do so explicitly
with the 'discharge method. This is recommended in favor of simply
losing their records.

The first checkup/treatment of 'attend! is synchronous in order
for component dependency management to be made as transparent
as possible. If you wish to skip this initial step, then you
may set the :skip-initial-checkup? option of attend! to true.


Example:

		(require '[com.stuartsierra.component :as component])
		(require '[component-clinic.core      :refer :all])

		(defrecord Unstable []
			component/Lifecycle
			(start [this]
				(prn :starting-everything)
				(-> this (assoc :health (atom nil))
						attend!))

			(stop [this]
				(prn :stopping-everything)
				(discharge! this))

			curable
			(treat! [this]
				(prn :treating!!!)
				(cond
				 (not @(:health this)) (reset! (:health this) 1)
				 :else                 (swap!  (:health this) inc)))

			diagnosable
			(healthy? [this]
				(prn :how-are-you-feeling-today?)
				(prn this)
				(let [conn @(:health this)]
					(and conn (> conn 3)))))

		(defn swait [this]
			(Thread/sleep (apply get-time [5 :seconds]))
			(component/stop this))

		(-> {:checkup-interval [1 :seconds]}
				map->Unstable
				component/start
				swait)

## Options

The 'attend! function accepts either just the component, or the component and some options.

The default options are below:

		{ :skip-initial-checkup? false
			:checkup-interval      [10 :seconds]
		}

## Logging

Component-Clinic uses taoensso.timbre for logging.

Various log-levels are used for different scenarios:

* Error: Exceptions thrown by your health? or treat! calls during the clinic loop
* Error: Component threw exception on admittance
* Warn:  Your component recovered
* Info:  Your component is aboiut to be treated
* Info:  Your component can't be checked as it doesn't implement diagnosable
* Info:  Admitting component
* Info:  Discharging component
* Trace: Your component is about to be checked
* Trace: Your component has been checked and found healthy

In general, an Info log-level will prevent you from missing any important messages.

## Crash Driven Development

In complex systems, if initialization becomes recovery, then you can
start your system in a crashed state, and let it recover. This has the advantage
of only requiring a piece of code for recovery, and no extra initialization
routine required.

Component-clinic helps facilitate this by allowing your component/Lifecycle
implementation to be kept absolutely minimal, and then allowing your curable
implementation to handle initialisation for you through recovery.

See <https://www.usenix.org/conference/hotos-ix/crash-only-software>.

## Contributing

If you wish to contribute to this codebase, you can fork and send a pull-request.

The project is built using travis-ci and git tags are released to clojars.
