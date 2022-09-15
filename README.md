# Hagall

This repository only contains Hagall releases as Hagall is not open source at this moment.

![](hagall.png)

_Hagall means Hail in Old Norse._

Hagall is a Real Time Networking Server responsible for processing, responding to and broadcasting networking messages to connected clients (participants) in a session similar to how a multiplayer networking engine handles message passing in an first-person-shooter game.

Hagall is through its module system extensible, but in its core a simple networking engine that manages 3 types of abstractions:

- **Session** - A session facilitates the communication and in-memory persistence of participants, entities and actions inside an OpenGL coordinate system in unit meters. A session is similar to an FPS game session. Participants' positional data and actions are sent and broadcast as quickly as possible and only the messages that are required to retrieve the current state of the session are stored in-memory to support late joiners. Multiple sessions can exist in the same Hagall server and each session is identified by a string ID in the format `<hagall_id>x<session_id>` e.g. `5fx3a`.
- **Participant** - Represents a connected client e.g. a mobile device or other hardware that wishes to interact with entities and other participants in a session.
- **Entity** - An entity is an object in a session with a _Pose_ and an ID and it is owned by a specific participant. Hagall does not care about what an entity represents, it could be a 3D asset, an audio source or a particle system. It's up to the application implementer to map their game objects to corresponding entities.

The core responsibilities of Hagall are:

- Creation and deletion of sessions; participant authentication and participant joining/leaving sessions.
- Addition and deletion of entities.
- Broadcasting of messages to participants.

# Server Operator's Manual

While we at Auki Labs run several Hagall servers in multiple regions on AWS, we allow for anyone to become a Hagall server operator and help us run part of our infrastructure.

At a later stage we plan to launch a system that rewards Hagall server operators with tokens based on traffic served.

If you have spare compute resources, enough bandwidth and wish to become a Hagall server operator, please see a deployment method below for instructions on how to get started.

## Running a Hagall server

### Minimum requirements

- x86 or ARM processor
- At least 64 MiB of RAM
- A stable Internet connection, we recommend at least 10 Mbps downstream and upstream
- A supported operating system, we currently provide pre-compiled binaries for Windows, macOS, Linux, FreeBSD, Solaris as well as Docker
- An HTTPS compatible web server or reverse proxy with an SSL certificate installed, alternatively use our Docker Compose or Kubernetes set-up that includes it

Auki's Hagall Discovery Service will perform regular checks on the health of your server to determine if it's fit to serve traffic. Make sure that you have enough spare compute capacity and bandwidth for hosting sessions or your server might be delisted from the network.

### Configuration

Configuration parameters can be passed to Hagall either as environment variables or flags.

`./hagall --public-endpoint https://hagall.example.com` (where `https://hagall.example.com` is the public, external address where this Hagall server is reachable) will launch Hagall with sane defaults, but if you for some reason want to modify the default configuration of Hagall you can run `./hagall -h` for a full list of parameters.

Here are some example parameters:

| Flag             | Environment variable   | Default | Sample                     | Description                                                                                                                |
| ---------------- | ---------------------- | ------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| --wallet-addr     | HAGALL_WALLET_ADDR     | _N/A_   | 0x0                        | The crypto wallet address to be rewarded. (currently N/A)                                                                  |
| --public-endpoint | HAGALL_PUBLIC_ENDPOINT | _N/A_   | https://hagall.example.com | The public endpoint where this Hagall server is reachable. This endpoint will be registered with Hagall Discovery Service. |
| --log-level       | HAGALL_LOG_LEVEL       | info    | debug                      | The log level (debug, info, warning or error)                                                                              |
## Binary

> **_NOTE:_** If you choose to use this deployment method, you need to set up your own HTTPS web server or reverse proxy with an SSL certificate.

### Currently supported platforms

We build pre-compiled binaries for these operating systems and architectures:

* Windows x86, x86_64
* macOS x86_64, ARM64 (M1)
* Linux x86, x86_64, ARM, ARM64
* FreeBSD x86, x86_64
* Solaris x86_64

1. Download the latest Hagall from [GitHub](https://github.com/aukilabs/hagall/releases)
2. Run it with `./hagall --public-endpoint=https://hagall.example.com`

You can optionally use a daemon manager such as systemd, launchd, SysV Init, runit or Supervisord.

## Docker

Hagall is available on [Docker Hub](https://hub.docker.com/r/aukilabs/hagall).

Here's an example of how to run it:

```
docker run -e HAGALL_PUBLIC_ENDPOINT=https://hagall.example.com aukilabs/hagall:stable
```

### Supported tags
_See the full list on [Docker Hub](https://hub.docker.com/r/aukilabs/hagall)._

* `latest` (bleeding edge, not recommended)
* `stable` (latest stable version, recommended)
* `v0` (specific major version)
* `v0.4` (specific minor version)
* `v0.4.14` (specific patch version)

### Docker Compose

Since Hagall needs to be exposed with an HTTPS address and Hagall itself doesn't terminate HTTPS, we recommend you to use our Docker Compose file that sets up an nginx proxy container that terminates HTTPS and a letsencrypt container that obtains a free Let's Encrypt SSL certificate alongside Hagall.

1. Download the latest Docker Compose YAML file from [GitHub](https://github.com/aukilabs/hagall/blob/main/docker-compose.yml)
2. Configure the environment variables to your liking (you must at least set `VIRTUAL_HOST`, `LETSENCRYPT_HOST` and `HAGALL_PUBLIC_ENDPOINT`)
3. With the YAML file in the same folder, start the containers using Docker Compose: `docker-compose up -d`

## Kubernetes

Auki provides a Helm chart for running Hagall in Kubernetes. We recommend that you use this Helm chart rather than writing your own Kubernetes manifests. For more information about [what Helm is](https://helm.sh/docs/topics/architecture/) and how to [install](https://helm.sh/docs/intro/install/) it, see Helm's official website.

### Requirements

* Kubernetes 1.14+
* Helm 3
* An HTTPS compatible ingress controller

### Installing

The chart can be deployed by CI/CD tools such as ArgoCD or Flux or it can be deployed using Helm on the command line like this:

```
helm repo add aukilabs https://charts.aukiverse.com
helm install hagall aukilabs/hagall --set config.HAGALL_PUBLIC_ENDPOINT=https://hagall.example.com
```

### Uninstalling

To uninstall (delete) the `hagall` deployment:

```
helm delete hagall
```

### Values

Please see [values.yaml](https://github.com/aukilabs/helm-charts/blob/main/charts/hagall/values.yaml) for the available values and their defaults.

Values can be overridden either by using a values file (the `-f` or `--values` flags) or by setting them on the command line using the `--set` flag. For more information, see the official [documentation](https://helm.sh/docs/helm/helm_install/).

You must at least set the `config.HAGALL_PUBLIC_ENDPOINT` key for server registration to work.


### Terms
*A. GENERAL TERMS
1. Term. This Agreement shall commence on the Effective Date and, unless terminated earlier, continue
until terminated.
2. Termination. Each party may terminate this Agreement at any time for any or no reason by providing
written notice to the other.
3. Survival. All terms that by their nature are intended to survive the expiration or termination of this
Agreement shall so survive.
4. Governing Law. This Agreement shall be governed by and construed in accordance with the laws of Hong
Kong without regard to any principle of conflicts of law.
5. Dispute Resolution. Any dispute, controversy or claim arising out of or relating to this Agreement, or the
breach termination or invalidity thereof, shall be settled by arbitration in accordance with the UNCITRAL
Arbitration Rules as at present in force and as may be amended by the rest of this Section; provided,
however, that Auki Labs may enforce its or its affiliates’ intellectual property rights in any court of competent
jurisdiction, including but not limited to injunctive relief. The appointing authority shall be Hong Kong
International Arbitration Centre. The place of arbitration shall be in Hong Kong at Hong Kong International
Arbitration Centre (HKIAC) and the arbitration proceedings shall be conducted in the English language.
There shall be only one arbitrator. Any such arbitration shall be administered by HKIAC in accordance with
HKIAC Procedures for Arbitration in force at the date of this Agreement including such additions to the
UNCITRAL Arbitration Rules as are therein contained.

B. LICENSE TERMS AND RESTRICTIONS
1. License for Software. Subject to the terms and conditions of this Agreement, Auki Labs hereby grants to
Company, and Company hereby accepts from Auki Labs, during the Term, the personal, non-exclusive, non-
transferable and non-assignable, royalty-free right, without the right to sublicense, to test, evaluate and run
the software specified in Exhibit A (“Software”) and to use the accompanying documentation (if any), for the
limited purpose of the internal evaluation by the Company only.
2. Authorized Use. Further to rights and license granted to Company hereunder, unless otherwise permitted
by Auki Labs, Company shall have a right to install the Software within one computer or other designated
hardware of (and controlled by) Company.
3. General Restrictions. Company shall not, and shall not permit any third party to: (i) use the Software
(including any associated documentation) for commercial or revenue generating purposes; (ii) copy,
translate, modify or make derivative works of any portion of the Software (including any associated
documentation); (iii) rent, disclose, publish, sell, assign, lease, pledge, lend, sublicense, market, transfer,
distribute or otherwise provide third parties access to any portion of the Software (including any associated
documentation); (iv) reverse engineer, decompile or disassemble the Software, or derive or attempt to derive
the source code, algorithmic nature or structure of any object code portions of the Software except and only
to the extent that such activity is expressly permitted by applicable law notwithstanding this limitation; (v) use
the Software (including any associated documentation) to create any product that competes with the
Software; (vi) remove or circumvent any protection or other restrictive technology mechanism of the
Software; (vii) remove or alter any proprietary markings or notices from the Software (including any
associated documentation), (viii) keep any archival copies of the Software, any copies or parts thereof,
except to the extent that applicable law notwithstanding this limitation expressly permits such; or (ix)

otherwise use any portion of the Software (including any associated documentation) in any manner not
expressly authorized in this Agreement.
4. Open Source Restrictions. The rights granted to Company do not include any license, right, power or
authority to subject the Software, in whole or in part, to Open Source License Terms. As used herein, Open
Source License Terms means terms in any license for software which require, as a condition of use,
modification and/or distribution of such software or other software incorporated into, derived from or
distributed with such software (a Work), any of the following: (a) the making available of source code or
design information regarding the Work; (b) the granting of permission for creating derivative works regarding
the Work; or (c) the granting of a royalty-free license to any party under intellectual property rights regarding
the Work. By means of example and without limitation, Open Source License Terms include the following
licenses or distribution models: (i) the GNU General Public License (GPL) or Lesser/Library GPL (LGPL), (ii)
the Artistic License (e.g. PERL), (iii) the Mozilla Public License, (iv) the Common Public License, (v) the Sun
Community Source License (SCSL), (vi) the Sun Industry Standards Source License (SISSL), (vii) the Sun
Industry Standards License (SISL), and (viii) the Open Software License.
5. Protection. Company shall take all reasonable measures necessary to protect the Software (including any
associated documentation) against mishandling, misappropriation and/or misuse by any party and shall
properly store the Software during the Term. Company shall notify Auki Labs immediately if Company learns
of any misappropriation, or unauthorized use or disclosure of the Software.
6. Marking. Company shall have the Software (including any associated documentation) properly marked as
being property of Auki Labs.
7. Reservation of Rights. This Agreement does not transfer any ownership interest in the Software or any
associated documentation. Except for those rights specifically to Company hereunder, Auki Labs, its
affiliates and their suppliers reserve all right, title and interest in and to the Software, the associated
documentation and their trademarks, trade names and service marks.
8. Injunctive Relief. Company acknowledges that if Auki Labs is required to bring an action to enforce the
provisions of this Agreement, the damages may be irreparable and difficult to measure and that Auki Labs
shall be entitled to seek injunctive relief in addition to any other relief available.
9. Feedback. If Company or any of its employees sends or transmits any communications or materials to
Auki Labs by mail, email, telephone, or otherwise, suggesting or recommending changes to the Software,
including without limitation, new features or functionality relating thereto, or any comments, questions,
suggestions, or the like(“Feedback”), Auki Labs is free to use such Feedback. Company hereby assigns to
Auki Labs its behalf, and on behalf of its employees, all right, title, and interest in, and Auki Labs is free to
use, without any attribution or compensation to Company, any ideas, knowhow, concepts, techniques, or
other intellectual property rights contained in the Feedback, for any purpose whatsoever, although Auki Labs
is not required to use any Feedback.

C. OBLIGATIONS
1. Company’s Obligations. Company shall: (a) refrain from making or publishing any information concerning
the Software without Auki Labs’ express prior written approval; and (b) promptly upon termination or
expiration of this Agreement, or at Auki Labs’ first request: i) return to Auki Labs all (copies of) the Software,
its associated documentation and any related materials, or if so requested by Auki Labs ii) erase and delete
from all of Company’s storage elements or devices all Software, including any associated documentation as
well as all copies thereof in whole or in part and in any form, and provide Auki Labs with its written
certification thereof.
2. Auki Labs Obligations. Auki Labs shall use reasonable commercial efforts to deliver to Company the
Software in a timely manner, via electronic transfer when possible.

E. CONFIDENTIALITY

1. Confidential Information. Auki Labs and its affiliates (Discloser) may disclose certain of its Confidential
Information to the Company or its affiliates (Recipient), directly or indirectly, in connection with this
Agreement. As used herein, Confidential Information means any confidential or proprietary information,
including designs, drawings, plans, formulae, algorithms, instructions, processes, programs, systems,
theories, specifications, techniques, tapes, disks, disk racks, models, data, flow charts, documentation,
processes, procedures, know-how, new product or technology information, prototypes, software (whether in
object code or source code), audio and or video bit streams, evaluation and benchmark results,
performance, manufacturing, development, or marketing techniques, development or marketing timetables,
business strategies and development plans, supplier information, personnel information, customer
information, pricing policies, quarterly reporting forms, financial information and any other information of a
similar nature, whether or not reduced to writing or other tangible form, and any other trade secret or non-
public business information which is: (a) marked as “confidential” or “proprietary” at the time of disclosure; or
(b) if disclosed orally, summarized and designated in writing as “confidential” or “proprietary” within thirty (30)
days from its disclosure; or (c) if not so marked or designated, should reasonably be considered confidential
by Recipient under the circumstances. For clarity: the Software and the related documentation and any
portions and derivatives thereof constitute Confidential Information of Auki Labs.
2. Confidentiality Obligations. Recipient shall, for a period of five (5) years from the date of disclosure, hold
Discloser’s Confidential Information in strict confidence and shall not disclose Discloser’s Confidential
Information (including not negotiate with or offer or agree to sell, lease or otherwise transfer any information
which includes or is based on Discloser’s Confidential Information) to any third party, except to such
employees, subcontractors or representatives of Recipient who clearly need to have access and who are
bound in writing by confidentiality obligations at least as protective as those set forth herein. Each Recipient
shall not use Discloser’s Confidential Information for any purpose other than as is strictly necessary to
exercise its rights or perform its obligations under this Agreement. Recipient shall exercise the same care as
it exercises to protect its own confidential and proprietary information of similar importance (but in no event
less than reasonable care). Each party shall be liable for any failure of its affiliates subcontractors and
representatives to abide by the provisions of this Agreement as if such failure was the act or omission of
such party.
3. Exceptions. Recipient’s confidentiality obligations shall not apply with respect to information to the extent
that such information can be shown, by written prior or contemporaneous evidence, to have been: (a) known
or available to the public prior to the date of disclosure to Recipient or to have become available to the public
thereafter through no unauthorized act or omission by Recipient; (b) rightfully in Recipient’s possession prior
to the date of disclosure to Recipient and not otherwise restricted as to disclosure; (c) disclosed to Recipient
without restriction by a third party who had a right to disclose and was not otherwise under an obligation of
confidence; or (d) independently developed by Recipient without use of or reference to Discloser’s
Confidential Information. In addition, each Recipient may disclose Confidential Information of Discloser to
the extent required by law or order of a court of competent jurisdiction, provided that, in such event,
Recipient shall promptly provide Discloser notice of such requirement enabling an intervention (and
Recipient shall cooperate with Discloser) to contest or minimize the scope of the disclosure.

F. DISCLAIMER; LIMITATION OF LIABILITY
1. Disclaimer. THE SOFTWARE AND ASSOCIATED DOCUMENTATION ARE PROVIDED ON AN “AS IS”
AND “WITH ALL FAULTS” BASIS AND THE ENTIRE RISK AS TO THE QUALITY, OR ARISING OUT OF
THE USE OR PERFORMANCE, OF THE SOFTWARE REMAINS WITH COMPANY. AUKI LABS HEREBY
EXPRESSLY DISCLAIMS ANY AND ALL WARRANTIES, WHETHER EXPRESS, IMPLIED OR
STATUTORY, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF NON-
INFRINGEMENT, MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.
2. Limitation of Liability. AUKI LABS SHALL NOT BE LIABLE TO COMPANY FOR ANY SPECIAL,
INDIRECT, CONSEQUENTIAL, PUNITIVE, OR INCIDENTAL DAMAGES (INCLUDING WITHOUT
LIMITATION DAMAGES FOR LOSS OF BUSINESS, BUSINESS INTERRUPTION, LOSS OF USE, LOSS
OF DATA OR INFORMATION, AND THE LIKE) ARISING OUT OF THIS AGREEMENT OR THE USE OF
OR INABILITY TO USE THE SOFTWARE, WHETHER OR NOT BASED ON TORT (INCLUDING

NEGLIGENCE), STRICT LIABILITY, BREACH OF CONTRACT, BREACH OF WARRANTY OR ANY
OTHER THEORY, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGES. EXCEPT FOR ANY
BREACH OF ITS CONFIDENTIALITY OBLIGATIONS, THE ENTIRE LIABILITY OF AUKI LABS
HEREUNDER AND COMPANY’S EXCLUSIVE REMEDY FOR ALL OF THE FOREGOING SHALL BE
LIMITED TO ACTUAL DAMAGES INCURRED BY COMPANY BASED ON REASONABLE RELIANCE UP
TO THE GREATER OF THE AMOUNT ACTUALLY PAID BY COMPANY HEREUNDER OR FIVE UNITED
STATES DOLLARS (USD 5.00), NOTWITHSTANDING ANY DAMAGES THAT COMPANY MIGHT INCUR
FOR ANY REASON WHATSOEVER. THE FOREGOING LIMITATIONS, EXCLUSIONS AND
DISCLAIMERS SHALL APPLY TO THE MAXIMUM EXTENT PERMITTED BY APPLICABLE LAW, EVEN IF
ANY REMEDY FAILS OF ITS ESSENTIAL PURPOSE.

G. MISCELLANEOUS
1. Relationship. Nothing in this Agreement shall create a joint venture, partnership or principal-agent
relationship between the parties.
2. Assignment. Company shall not assign or transfer any of its rights or obligations hereunder without the
prior written consent of Auki Labs. Subject to the foregoing, this Agreement shall be binding upon and inure
to the benefit of the parties and their respective successors and permitted assigns. Any attempted
assignment other than in strict compliance with this Section shall be void.
3. Notices. All notices required or permitted hereunder shall be in writing and shall be deemed delivered
upon hand delivery, upon receipt if by acknowledged facsimile communication, or upon receipt if sent by
world renown overnight courier or mailed by registered or certified mail, return receipt requested, postage
prepaid, addressed to a party at its address set forth above or such other address of which a party may
notify the other party from time to time.
4. Waiver. A waiver of any right hereunder shall in no way waive any other rights. No waiver, alteration,
modification or amendment of this Agreement shall be effective unless in writing and signed by both parties.
5. Severability. In the event that any provision of this Agreement is held to be invalid, illegal or
unenforceable, such provision shall be deemed amended to achieve the economic effect of the intent of the
parties in a valid, lawful and enforceable manner, or if not possible, the deleted and ineffective to the extent
thereof, without affecting any other provision of the Agreement.
6. Entire Agreement. This Agreement constitutes the entire agreement regarding the subject matter hereof
and supersedes all prior agreements, understandings and communications, oral and written, between the
parties regarding the subject matter hereof.
7. Execution. This Agreement may be executed in counterparts (and may be exchanged by fax or e-mail
when signed), each of which shall be deemed to be an original, and all of such counterparts shall together
constitute one instrument.

IN WITNESS WHEREOF, duly authorized representatives of each party have executed this Agreement as of
the Effective Date:*
