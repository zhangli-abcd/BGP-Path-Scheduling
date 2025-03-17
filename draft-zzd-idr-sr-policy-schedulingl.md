---
title: BGP SR Policy Extensions for Path Scheduling
abbrev: BGP SR Policy Scheduling
docname: draft-zzd-idr-sr-policy-scheduling-latest
obsoletes:
updates:
date:
category: std
submissionType: IETF

ipr: trust200902
area: General
workgroup: IDR
keyword: Internet-Draft

author:
 -
  ins: L. Zhang
  name: Li Zhang
  organization: Huawei
  email: zhangli344@huawei.com
  role: editor
  street: Beiqing Road
  city: Beijing
  country: China
 -
  ins: T. Zhou
  name: Tianran Zhou
  organization: Huawei
  email: zhoutianran@huawei.com
 -
  ins: J. Dong
  name: Jie Dong
  organization: Huawei
  email: jie.dong@huawei.com
 -
  ins: M. Wang
  name: Minxue Wang
  organization: China Mobile
  email: wangminxue@chinamobile.com
 -
  ins: N. Nzima
  name: Nkosinathi Nzima
  organization: MTN
  email: Nkosinathi.Nzima@mtn.com

normative:

informative:
 
...

--- abstract

Segment Routing (SR) policy enables instantiation of an ordered list of segments with a specific intent for traffic steering. When using SR policy in a time-variant network, delivering the time-variant information associated with paths is necessary in some scenarios.

This document proposes extensions to BGP SR Policy to deliver the schedule information of candidate path (segment list) and its associated attributes.

--- middle

# Introduction {#intro}

{{?RFC9657}} introduces a set of time-variant network use cases where the topology of the network changes predictably. When the networks uses traditional routing protocols, it takes these topology changes as unexpected events and may cause and packet loss. However, the topology changes of these networks can be predicted in advance, therefore some measures can be taken in advance to prevent the packet loss. With this idea, {{!I-D.ietf-tvr-requirements}} describes the requirements of using the time-variant information in a network. In {{Section 3.4.1 of !I-D.ietf-tvr-requirements}}, it describes the centralized routing scenarios with time-variant information, in which the network entities receive the time variable information and traffic forwarding rules directly from a logically centralized source(an Orchestrator or network controller). The time-variant information is especially essential when there is a risk that a logically centralized source may loses connectivity with the network entities.

Segment Routing (SR) policy {{!RFC9256}} is a set of candidate SR paths consisting of one or more segment lists and necessary path attributes. It describes the traffic forwarding rules for specific flows. {{!I-D.ietf-idr-sr-policy-safi}} introduces how BGP may be used to distribute SR Policy candidate paths. It introduces a BGP SAFI with new NLRI to advertise the candidate paths and specific attributes of a SR Policy from a controller or path computation engine (PCE) to the network entities. However, when using BGP SR Policy in a time-variant network, it can't advertise the schedule information associated with paths.

This document proposes extensions to BGP SR Policy to carry the schedule information of candidate paths/segment lists.

## Requirements Language

{::boilerplate bcp14-tagged}

# Motivation 

Most of the time-variant network use cases using BGP SR Policy could be benefit from this work. In some cases, carrying the time-variant information with SR Policy is essential.

This section describes the cases that requires extending SR Policy with schedule information.

## Network with Discontinuous Links

In some time-variant network cases, the links between the network entities and network controller may very weak or intermittent, this is very typical in Resource Preservation and Dynamic Reachability network{{?RFC9657}}. In these cases, Real-time SR policy advertising (before changes occur) may not be timely. For example, when a link of an old path is about to be disconnected, the network controller is going to advertise a new path to the headend. However, the link between the headend and the network controller is not available. As a result, the new path cannot be advertised in time, causing packet loss.

Therefore, in these cases, once the links between the headend and network controller are available, the controller need to advertise the paths with schedule information for a period in the future to the headend. Then the headend could determine valid paths in the future based on the schedule information of SR policy.

## Network with Frequent Topology Changes

There are also some time-variant network cases that topology changes frequently. This is very typical when the number of network entities is very large (For example, a Dynamic Reachability network with hundreds or thousands of nodes). In this kind of time-variant network, a path form one network entity to another changes frequently, sometimes it can only be maintained for a few minutes or seconds.

Considering that there are multiple paths in a network that computed by the controller, the SR Policies with candidate paths may be advertised to the headend every few seconds. It poses great changeling to the network controller. However, using schedule information could advertise several paths once, which greatly mitigate the pressure of network controllers.

# Schedule Information in SR Policy

The NLRI defined in {{!I-D.ietf-idr-segment-routing-te-policy}} contains the SR Policy candidate path. The content of the SR Policy Candidate Path is encoded in the Tunnel Encapsulation Attribute defined in {{?RFC9012}} using a new Tunnel-Type called SR Policy Type with codepoint 15. The SR Policy encoding structure is as follows:

~~~
SR Policy SAFI NLRI: <Distinguisher, Policy-Color, Endpoint>
      Attributes:
         Tunnel Encapsulation Attribute (23)
            Tunnel Type: SR Policy (15)
                Binding SID
                SRv6 Binding SID
                Preference
                Priority
                Policy Name
                Policy Candidate Path Name
                Explicit NULL Label Policy (ENLP)
                Segment List
                    Weight
                    Segment
                    Segment
                    ...
                ...
~~~

A candidate path includes multiple SR paths, each of which is specified by a segment list. The schedule information can be applied to a candidate path, indicating the valid time for each candidate path and its associated attributes.
The new SR Policy encoding structure is expressed as below:

~~~
SR Policy SAFI NLRI: <Distinguisher, Policy-Color, Endpoint>
      Attributes:
         Tunnel Encapsulation Attribute (23)
            Tunnel Type: SR Policy (15)
                Binding SID
                SRv6 Binding SID
                Preference
                Priority
                Policy Name
                Policy Candidate Path Name
                Explicit NULL Label Policy (ENLP)
                Schedule Information
                Segment List
                    Weight
                    Segment
                    Segment
                    ...
                ...
~~~

The schedule Information also can be applied to a segment list to indicate the valid time for a segment list and its associated attributes.
The new SR Policy encoding structure is expressed as below:

~~~
SR Policy SAFI NLRI: <Distinguisher, Policy-Color, Endpoint>
      Attributes:
         Tunnel Encapsulation Attribute (23)
            Tunnel Type: SR Policy (15)
                Binding SID
                SRv6 Binding SID
                Preference
                Priority
                Policy Name
                Policy Candidate Path Name
                Explicit NULL Label Policy (ENLP)
                Segment List
                    Schedule Information
                    Weight
                    Segment
                    Segment
                    ...
                ...
~~~

# Schedule Information Sub-TLV

The schedule information sub-TLV indicates one or more valid time slot for one or more paths. The format of schedule information sub-TLV is shown as follows:

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |     Length    |Schedule Number|    Reserved   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
/                        Schedules                              /
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #ref-to-fig1  title="Scheduling Information Sub-TL"}

Type: TBD1

Length: the size of the value field in octets.

Schedule Number: indicates the number of schedules.

Schedules: one or more schedules, each schedule indicates the duration when the candidate path (segment list) is active. The format of each schedule is shown as follows:

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           Schedule-id                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Flags  |S|P|R|    Length     |          Reserved             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          Start Time                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Start Time(Continue)                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       End Time/Duration                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                  End Time/Duration(Continue)                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   Recurrence count/Bound(Optional)            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   Recurrence count/Bound(Optional)            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Frequency (Optional)                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

~~~
{: #ref-to-fig2  title="Format of Schedules"}

Schedule-id: 32-bit value, the unique identifier to distinguish each schedule within a SR Policy, this value is allocated by the SR Policy generator.

Flags: 8 bits, currently only 2 bits are used, the other bits are reserved.

Length: 8 bits, indicates the length of this schedule in octets.

S (Schedule type): one-bit flag to indicate the type of a schedule. If S=0, it indicates the schedule only has one instance, the Recurrence Count/Bound, Frequency and Interval field should not be included in the sub-TLV; If S=1, it indicates the schedule has multiple instances, the Recurrence Count/Bound, Frequency and Interval field should be included.

P (Period type): one-bit flag to indicate the description type of a period. if P=1, then the period is described by a start time filed and an end time field; If P =0, then the period is described by a start time field and a duration time field.

R (Recurrence bound type): one-bit flag to indicate the how to determine whether the recurrence is end. if R=1, then the end of recurrence is determined by a detail timepoint; If R = 0, then the end of the recurrence is determined by the number of occurrences.

Start Time: 64-bit value, the number of seconds since the epoch, it indicates when the candidate path (segment list) and its associated attributes start to take effect.

End Time (Duration): 64-bit value, if the flag P=1, then it is the number of seconds since the epoch, it indicates when the candidate path (segment list) and its associated attributes becomes ineffective. If the flag P=0, then it is the number of seconds since the Start Time, it indicates how long the candidate path (segment list) and its associated attributes are effective.

Recurrence Count/Bound(optional): 64-bit value, this field SHOULD be included when the flag P is set to 1. When the flag R=0, then this field indicates the max number of occurrences. For example, if it is set to 2, then the schedule will repeat twice with the specified Frequency and Interval. When the flag R=1, then tis field indicates the bounded timepoint of recurrence, it is descripted by the number of seconds since the epoch.

Frequency(optional): 32-bit value, this field should be included when the flag S is set to 1. It is the numbers of seconds since the Start Time of an instance to the Start Time of next instance. This field indicates the recurrence frequency for all the instance of this schedule.

# Operations

## Advertisement of SR Policies with Schedule Information

As described in {{Section 4.1 of ?I-D.ietf-idr-sr-policy-safi}}, typically, an SR Policy is computed by a controller or a path computation engine (PCE) and originated by a BGP speaker on its behalf. The schedule information is also computed by a controller or a PCE which can access to all the predicted topology changes. The predicted topology changes could be got from management interfaces or other means.

The controller or PCE SHOULD maintain a time-variant event database as described in {{Section 6 of !I-D.zdm-tvr-applicability}} to store all the predicted information, and compute SR Policies with schedule information based on the database.

Each Candidate Path or Segment List may have multiple schedules, each schedule is identified by schedule-id, the controller or PCE MUST make the schedule-id unique within a specific SR Policy.

The controller or PCE MUST ensure that the BGP speakers who will receive the SR Policy with schedule information have the capability to deal with the schedule information.

## Reception of an SR Policy with Schedule Information

### Validation of Schedule Information

On reception of an SR Policy, a BGP speaker first determines if it is valid as described in {{Section 4.2.1 of !I-D.ietf-idr-sr-policy-safi}}. When a headend receives a SR Policy form it neighbors or controller, it SHOULD perform schedule information validation based on the following rules:

- The headend MUST NOT use the SR Policy with schedule information when it does not have the capability to deal with the schedule information.

- When multiple schedules are present within one SR Policy, the schedule-id of each schedules MUST be different. If there are multiple schedules with the same schedule-id, then the SR Policy MUST be considered as malformed.

- The Start Time of Schedule Information TLV MUST be later than the time it be received, if not, the SR Policy MUST be considered as malformed.

- If the End Time/Duration field is used to indicated the end time, then it MUST be later than the Start Time, if not, the SR Policy MUST be considered as malformed.

- If the Frequency field is present in the Schedule Information TLV, then it MUST be greater than the difference between the End Time and Start Time(P=1), or greater than the Duration(P=0), if not, the SR Policy MUST be considered as malformed.

- If the Recurrence Count/Bound field is present and used to indicate the boundary time point, then it MUST be greater than the End Time(P=1), or greater than the sum of Start Time and Duration(P=0), if not, the SR Policy MUST be considered as malformed.

If a headend receives a SR Policy that considered as malformed, then the SR Policy MUST NOT be used.

As described in {{!I-D.ietf-idr-sr-policy-safi}}, the BGP SR Policy is distinguished by <distinguisher, color, endpoint> tuple. Whenever a headend receives a SR Policy that it has received before, the SR Policy is considered as the fully replacement of the old one.

### Enable/Disable Candidate Paths/Segment Lists

When a headend receives a SR Policy with schedule information, it SHOULD parse the SR Policy and save the schedule information locally. When a data packet arrives, it will steer it to a specific SR Policy by color or other means.

Within a specific SR Policy, the headend dynamically enable/desable different Candidate Paths/Segment Lists based on the schedule information.

# Security Considerations

TBD

# IANA Considerations
This document defines a sub-TLV in the registry "BGP Tunnel Encapsulation Attribute sub-TLVs" to be assigned by IANA:

| Value | Description | Reference |
|-------|-------------|-----------|
| TBD1 | Schedule Information (SI) | This document |

The type value of this sub-TLV is recommended to be token from the range of 64-125(First Come First Served).


--- back
