# Speaker script – CPATS, SOICT 2026 (target 10–11 min)

About 1,404 words; at a calm 125–130 words per minute this is about 11 minutes. The session slot is 15 minutes including Q&A.

## Slide 1. Title  (~37 s)

Good morning everyone. My name is Bao Tran, from the School of Information and Communication Technology, Hanoi University of Science and Technology. This is joint work with my supervisor, Dr. Phuong Khanh Nguyen.

Today I will present CPATS, a cooperative parallel adaptive tabu search for truck–drone routing.

## Slide 2. Motivation: time-critical specimen transport  (~52 s)

Our motivating application is the transport of biological samples. Specimens collected at clinics and health centers have to reach a central laboratory before they degrade.

Ground vehicles have large capacity and can reach every site. Drones are fast and fly directly, but they can carry only light items and have limited endurance. A mixed fleet combines both strengths.

The operational question is: whether each site is served by a truck or a drone, how the sites are grouped into trips, how those trips are shared among the vehicles, and in what order the sites are visited. And because every sample matters, we want the last sample to arrive as early as possible. In other words, we minimize the makespan.

## Slide 3. The MMVRPD-MTV problem  (~52 s)

This setting is formalized as the MMVRPD-MTV, introduced by Nguyen et al. at SOICT 2025.

Each customer is served by exactly one eligible vehicle, and some customers can only be served by ground vehicles. Trucks and drones run independently, with no synchronization, and every vehicle can perform several trips from the depot.

The constraints are vehicle capacity, drone endurance, and a maximum waiting time between pickup and return to the depot.

Solutions that assign and order customers very differently can have almost the same makespan. The makespan alone then gives little guidance on where to search, so it pays to keep several structurally different solutions instead of converging on one.

## Slide 4. From sequential search to cooperation  (~51 s)

The existing solver for this problem is a sequential adaptive tabu search, S-ATS, which follows a single trajectory.

Running several S-ATS trajectories in parallel, independent multistart, widens the search, but each trajectory stays blind to what the others find. Cooperative multi-search lets trajectories exchange information while they run. To our knowledge, no truck–drone routing study has used cooperative multi-search in which concurrent trajectories exchange complete solutions through a shared elite pool.

So we ask three research questions. RQ1: how much does parallel multistart improve on S-ATS? RQ2: does cooperation add value beyond parallelism alone? RQ3: which parts of the exchange policy contribute to that value?

## Slide 5. CPATS: asynchronous master–worker architecture  (~54 s)

CPATS uses an asynchronous master–worker architecture.

Each of the p workers runs a complete S-ATS trajectory, with its own current solution, personal best, tabu list, adaptive weights, and random stream. The master performs no neighbourhood search at all. It only manages a small, bounded elite pool.

Workers interact with the master in two ways. PUSH submits a solution and does not block. PULL requests an elite and suspends only the requesting worker until the master replies. So there is no global synchronization barrier.

The master–worker architecture and elite memory are not new in themselves. Our contribution is the exchange policy: what to send, what to keep, what to give back, and when to use it.

## Slide 6. The exchange policy: a four-step cooperation cycle  (~79 s)

The policy is a cycle of four steps.

First, submit. A worker pushes its personal best only when it improves the last submitted one by at least 0.1 percent, which avoids flooding the pool with near-duplicates.

Second, maintain. When the pool is full, a newcomer competes only with its structurally closest incumbent, and replaces it only if it has a smaller makespan.

Third, retrieve. A worker asks for help after four segments without improvement, and never gets its own submissions back. The master ranks the elites by makespan and picks one at random, giving the best one the highest chance and lowering the chance step by step down the ranking. With four elites, the chances are 40, 30, 20 and 10 percent.

Fourth, accept. The worker decides. It accepts an elite that beats its personal best. Otherwise the elite must be at most one percent worse than the worker's personal best, and far enough from its current solution, which I explain on the next slide.

An accepted elite restarts the search with an empty tabu list, but the learned weights and penalties are kept.

## Slide 7. One distance, two uses: the EDGE distance  (~91 s)

The EDGE distance is one minus the overlap of the directed edges used by two solutions. Depot edges are included, so it also reflects how customers are grouped into trips, and it is computed in linear time.

On the master side, the distance decides which pool solution a newcomer may replace. When the pool is full, a newcomer is compared only with its closest incumbent, and replaces it only if it has a smaller makespan. Improvements can therefore survive in several structural regions. There is no hard distance guarantee, but in calibration the pool diversity rises from 0.07 with quality-only replacement to 0.61.

On the worker side, it drives selective acceptance. Together with the elite, the master returns a threshold: the median distance between the worker's current solution and the elites it could have received. A non-improving elite is accepted only if it is at most one percent worse than the worker's personal best, and at least this far from its current solution. Because the threshold is computed for each request, it adapts to how spread out the pool is at that moment. Compared with always accepting, this improves the final deviation from minus 0.50 to minus 0.69 percent.

## Slide 8. Experimental design  (~55 s)

For the experiments, all methods were implemented in C++ and run on an Intel Core i5-14400F. S-ATS uses one performance core. IMS-6 and CPATS use six workers, each pinned to its own performance core, and the CPATS master runs on an efficiency core, with its overhead counted inside the time limit.

We use the 128 benchmark instances of Nguyen et al. Instances with up to 20 customers are solved identically by all methods, and four instances are reserved for calibration only. That leaves 76 held-out instances with 50 to 1000 customers.

The time limit for each size was fixed from sequential runs before any parallel method was evaluated, and every method was run 10 times per instance.

## Slide 9. Staged calibration: each choice against its baseline  (~43 s)

This slide addresses RQ3: which parts of the exchange policy contribute.

The mechanism was calibrated in five stages on the four calibration instances only, changing one component at a time against a baseline.

In stage A, compared with pushing every new best, the 0.1-percent threshold cuts push events from 1.82 to 0.77 per million evaluations, a 58 percent reduction, while the anytime measure IRPD stays almost the same, 3.25 versus 3.27. Stages B and C are the two mechanisms from the previous slide: selective acceptance moves the final deviation from minus 0.50 to minus 0.69, and EDGE crowding raises pool diversity from 0.07 to 0.61. In stages D and E, four stalled segments and a one-percent acceptance tolerance both reach minus 0.69, better than the neighbouring values at around minus 0.55. Communication stays very light, about 0.02 pulls per million evaluations.

## Slide 10. Main result: cooperation beats parallel multistart  (~55 s)

Now the main result, on the 76 held-out instances. Negative values mean that we improve on the published best-known solutions.

Sequential S-ATS ends at plus 0.39 percent. Six independent workers bring this down to minus 0.38 percent. That answers RQ1.

CPATS reaches minus 0.73 percent, a further 0.35 percentage points from cooperation alone, with the same workers and the same time budget. That answers RQ2.

CPATS wins on 62 instances, ties on 12, and loses only 2 against IMS-6. A sign test gives a p-value below ten to the minus fifteen. The gain is positive in every size group, including sizes 50, 100, and 1000, which were never seen during calibration, and it is largest for 200 customers.

## Slide 11. Anytime performance  (~27 s)

Cooperation also helps throughout the run, not only at the end.

This plot shows the mean best-so-far deviation over the time budget. Lower is better, and the zero line is the level of the published best-known solutions. CPATS is the best at every checkpoint. Its curve is already below zero at the 50 percent checkpoint, while IMS-6 gets there only at 62.5 percent. Sequential S-ATS stays above zero even at the end, so on average it does not reach the published best-known solutions.

## Slide 12. Conclusions and future work  (~56 s)

To conclude. We proposed CPATS, a cooperative multi-search in which parallel tabu search trajectories exchange complete solutions through a shared elite pool, which to the best of our knowledge is new for truck–drone routing. It improves the published best-known solutions by 0.73 percent on average, and beats independent multistart by 0.35 points with the same resources.

The main limitations are a calibration on only four instances, and runs with six workers on one machine, so scalability and a possible master bottleneck remain open. The EDGE distance also ignores service modes.

Next, we plan factorial calibration, larger worker counts with distributed or hierarchical pools, and island-model variants that we are developing. The instances and improved solutions are available on GitHub.

## Slide 13. Thank you  (~7 s)

Thank you very much for your attention. I am happy to take your questions.

## Likely questions (short answers)

**Q: Does CPATS scale beyond six workers? Is the master a bottleneck?**
We only tested p = 6 on one machine, which we list as a limitation. Communication is very light, under one PUSH and about 0.02 PULL per million evaluations, so we do not expect the master to be a bottleneck at moderate p. Larger p and distributed or hierarchical pools are future work.

**Q: Why does the EDGE distance ignore service modes?**
It is cheap (linear time) and is called on every PUSH and PULL. Depot edges already capture trip composition. A mode-labeled edge set is a cheap extension we plan to test. The richer multi-level distance is used only as an a-posteriori diagnostic.

**Q: Why did you stop at δ = 0.036?**
The grid was fixed before the staged calibration. Larger pools are a natural next step and are part of the planned factorial calibration.

**Q: How robust is the 0.35-point gain?**
CPATS wins on 62 of 76 instances, ties on 12, and loses on 2. A two-sided sign test gives p < 10⁻¹⁵, and the gain is positive in every size group, including sizes not used for calibration.

**Q: Is the comparison fair?**
IMS-6 and CPATS use the same six workers and the same wall-clock limits, and the master's overhead counts inside the limit. The time limits were fixed from S-ATS runs before any parallel method was evaluated.
