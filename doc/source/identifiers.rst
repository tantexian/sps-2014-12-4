..
      Copyright 2010 OpenStack Foundation
      All Rights Reserved.

      Licensed under the Apache License, Version 2.0 (the "License"); you may
      not use this file except in compliance with the License. You may obtain
      a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

      Unless required by applicable law or agreed to in writing, software
      distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
      WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
      License for the specific language governing permissions and limitations
      under the License.

Image Identifiers
=================

demos are uniquely identified by way of a URI that
matches the following signature::

  <sps Server Location>/v1/demos/<ID>

where `<sps Server Location>` is the resource location of the sps service
that knows about an image, and `<ID>` is the image's identifier. Image
identifiers in sps are *uuids*, making them *globally unique*.
