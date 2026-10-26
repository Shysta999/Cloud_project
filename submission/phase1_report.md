# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
50
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->248.1<!--/answer--> | <!--answer:vm_warm_median_ms-->211.3<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->1707.2<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->631.7<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->2094.2<!--/answer--> | <!--answer:function_warm_median_ms-->636.0<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
Phase 1 — deployment measurements
------------------------------------------------------------
[OK  ] vm        cold=  248.1ms  warm_median=  211.3ms
            #############
[OK  ] cloudrun  cold= 1707.2ms  warm_median=  631.7ms
            ########################################
[OK  ] function  cold= 2094.2ms  warm_median=  636.0ms
            ########################################
------------------------------------------------------------
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
*VM (IaaS)*: nothing is created automatically. A single e2-micro machine absorbs all the traffic; the hard limit is the size of the machine I chose (2 shared vCPUs, 1 GB of RAM). Beyond that, requests slow down or fail.
*Cloud Run*: new container instances are created on demand, and scale back down to zero when there is no traffic. The limit comes from the flag I passed: `--max-instances=3`.
*Cloud Function*: new function instances are created automatically on demand (they run on Cloud Run infrastructure under the hood), then disappear when traffic drops. The limit is the platform's default max-instances, since I did not set one..
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
*VM*: It is the heaviest. I created the machine, opened the firewall, installed Python and the dependencies, copied the code, started the server and configured it to restart automatically. OS updates, security and monitoring remain my responsibility.
*Cloud Run*: I wrote a Dockerfile, built the image with Cloud Build, pushed it to Artifact Registry, then deployed with a single command. No OS to manage, but I have to maintain the Dockerfile.
*Cloud Function*: It is the lightest. I only provided the code, with no Dockerfile and no server to manage: Google handles the runtime and execution.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-571810918177.us-central1.run.app
LAST DEPLOYED BY: service-571810918177@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-26T19:13:48.404919Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-571810918177.us-central1.run.app
LAST DEPLOYED BY: yashica.mojee@eurecom.fr
LAST DEPLOYED AT: 2026-10-26T18:54:56.083588Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output shows two Cloud Run services: `echo-bot`, which I deployed myself, and `echo`, which is my Cloud Function. It appears in the Cloud Run list because a 2nd-gen Cloud Function is actually a Cloud Run service managed by Google; it was deployed by the `gcf-admin-robot` service account, not by me. That explains why the cold starts are so similar: both are containers on the same infrastructure, started the same way when no instance is warm. What differs is the deployment path. For Cloud Run, I wrote the Dockerfile, built the image and pushed it myself. For the Function, I only gave the code, and Google generated the container image, the server and the runtime for me.
<!--/answer-->
