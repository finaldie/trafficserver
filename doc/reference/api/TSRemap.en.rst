.. Licensed to the Apache Software Foundation (ASF) under one
   or more contributor license agreements.  See the NOTICE file
   distributed with this work for additional information
   regarding copyright ownership.  The ASF licenses this file
   to you under the Apache License, Version 2.0 (the
   "License"); you may not use this file except in compliance
   with the License.  You may obtain a copy of the License at
   
       http://www.apache.org/licenses/LICENSE-2.0
   
   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

.. default-domain:: c

===========
TSRemapInit
===========

Synopsis
========

`#include <ts/ts.h>`
`#include <ts/remap.h>`

.. function:: TSReturnCode TSRemapInit(TSRemapInterface * api_info, char* errbuf, int errbuf_size)
.. function:: void TSRemapDone(void)
.. function:: TSRemapStatus TSRemapDoRemap(void * ih, TSHttpTxn rh, TSRemapRequestInfo * rri)
.. function:: TSReturnCode TSRemapNewInstance(int argc, char * argv[], void ** ih, char * errbuf, int errbuf_size)
.. function:: void TSRemapDeleteInstance(void * )
.. function:: void TSRemapOSResponse(void * ih, TSHttpTxn rh, int os_response_type)

Description
===========

The Traffic Server remap interface provides a simplified mechanism for
plugins to manipulate HTTP transactions. A remap plugin is not global; it
is configured on a per-remap rule basis, which enables you to customize
how URLs are redirected based on individual rules in the remap.config
file. Writing a remap plugin consists of implementing one or more of the
remap entry points and configuring the remap.config configuration file to
route the transaction through your plugin. Multiple remap plugins can be
specified for a single remap rule, resulting in a remap plugin chain
where each pligin is given an opportunity to examine the HTTP transaction.

:func:`TSRemapInit` is a required entry point. This function will be called
once when Traffic Server loads the plugin. If the optional :func:`TSRemapDone`
entry point is available, Traffic Server will call then when unloading
the remap plugin.

A remap plugin may be invoked for different remap rules. Traffic Server
will call the entry point each time a plugin is specified in a remap
rule. When a remap plugin instance is no longer required, Traffic Server
will call :func:`TSRemapDeleteInstance`.

:func:`TSRemapDoRemap` is called for each HTTP transaction. This is a mandatory
entry point. In this function, the remap plugin may examine and modify
the HTTP transaction.

Return values
=============

:func:`TSRemapInit` and :func:`TSRemapNewInstance` should return
:data:`TS_SUCCESS` on success, and :data:`TS_ERROR` otherwise. A
return value of :data:`TS_ERROR` is unrecoverable.

:func:`TSRemapDoRemap` returns a status code that indicates whether
the HTTP transaction has been modified and whether Traffic Server
should continue to evaluate the chain of remap plugins. If the
transaction was modified, the plugin should return
:data:`TSREMAP_DID_REMAP` or :data:`TSREMAP_DID_REMAP_STOP`; otherwise
it should return :data:`TSREMAP_NO_REMAP` or :data:`TSREMAP_NO_REMAP_STOP`.
If Traffic Server should not send the transaction to subsequent
plugins in the remap chain, return :data:`TSREMAP_NO_REMAP_STOP`
or :data:`TSREMAP_DID_REMAP_STOP`.  Returning :data:`TSREMAP_ERROR`
causes Traffic Server to stop evaluating the remap chain and respond
with an error.

See also
========

:manpage:`TSAPI(3ts)`
