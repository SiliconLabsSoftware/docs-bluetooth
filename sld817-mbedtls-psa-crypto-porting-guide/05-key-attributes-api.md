# Key Attributes API

The following table lists the PSA Crypto API for the [key attributes](./04-key-management-in-psa-crypto.md#key-management-in-psa-crypto).

<table class="classic">
  <colgroup>
    <col width="30%">
    <col width="auto">
  </colgroup>
  <thead>
    <tr>
      <th><strong>API</strong></th>
      <th><strong>Description</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>psa_key_attributes_init(…)</code></p></td>
      <td><p>Initialize the key attributes (<code>psa_key_attributes_t</code>) before calling any function.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_get_key_attributes(…)</code></p></td>
      <td><p>Retrieve the key attributes (<code>psa_key_attributes_t</code>) of a key if successful.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_reset_key_attributes(…)</code></p></td>
      <td><p>Reset the key attributes (<code>psa_key_attributes_t</code>) to an initialized state.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_set_key_type(…)</code></p></td>
      <td><p>Declare the key type (<code>psa_key_type_t</code>) of a key.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_get_key_type(…)</code></p></td>
      <td><p>Retrieve the key type (<code>psa_key_type_t</code>) from key attributes.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_set_key_bits(…)</code></p></td>
      <td><p>Declare the key size (<code>psa_key_bits_t</code>) of a key.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_get_key_bits(…)</code></p></td>
      <td><p>Retrieve the key size (<code>size_t</code>) from key attributes.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_set_key_usage_flags(…)</code></p></td>
      <td><p>Declare the usage flags (<code>psa_key_usage_t</code>) for a key.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_get_key_usage_flags(…)</code></p></td>
      <td><p>Retrieve the usage flags (<code>psa_key_usage_t</code>) from key attributes.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_set_key_algorithm(…)</code></p></td>
      <td><p>Declare the permitted algorithm policy (<code>psa_algorithm_t</code>) for a key.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_get_key_algorithm(…)</code></p></td>
      <td><p>Retrieve the algorithm policy (<code>psa_algorithm_t</code>) from key attributes.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_set_key_id(…)</code></p></td>
      <td><p>Declare a key as persistent and set its key identifier (<code>psa_key_id_t</code>).</p></td>
    </tr>
    <tr>
      <td><p><code>psa_get_key_id(…)</code></p></td>
      <td><p>Retrieve the key identifier (<code>psa_key_id_t</code>) from key attributes.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_set_key_lifetime(…)</code></p></td>
      <td><p>Set the location (<code>psa_key_lifetime_t</code>) of a persistent key.</p></td>
    </tr>
    <tr>
      <td><p><code>psa_get_key_lifetime(…)</code></p></td>
      <td><p>Retrieve the lifetime (<code>psa_key_lifetime_t</code>) from key attributes.</p></td>
    </tr>
  </tbody>
</table>

The following sections describe how to use the key attributes API to set up the [storage](./04-key-management-in-psa-crypto.md#) for a key. Refer to the quick reference examples in [Symmetric Key](./06-migration-guide.md#symmetric-key) and [Asymmetric Key](./06-migration-guide.md#asymmetric-key) for more details.

**Volatile Plain Key**

<table class="classic">
  <colgroup>
    <col width="20%">
    <col width="30%">
    <col width="20%">
    <col width="30%">
  </colgroup>
  <thead>
    <tr>
      <th><strong>Key ID</strong></th>
      <th><strong>Persistence Level</strong></th>
      <th><strong>Location Indicator</strong></th>
      <th><strong>API Flow</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>= 0</td>
      <td><code>PSA_KEY_PERSISTENCE_VOLATILE</code></td>
      <td>Local (0x0)</td>
      <td>
        It is the default setting after calling <code>psa_key_attributes_init()</code>.
        No need to call <code>psa_set_key_id()</code> and <code>psa_set_key_lifetime()</code>.
      </td>
    </tr>
  </tbody>
</table>

**Example**:

```sh
psa_key_attributes_t key_attr;
key_attr = psa_key_attributes_init();
psa_set_key_type(&key_attr, PSA_KEY_TYPE_ECC_KEY_PAIR(PSA_ECC_FAMILY_SECP_R1));
psa_set_key_bits(&key_attr, 256);
psa_set_key_usage_flags(&key_attr, PSA_KEY_USAGE_SIGN_HASH | PSA_KEY_USAGE_VERIFY_HASH);
psa_set_key_algorithm(&key_attr, PSA_ALG_ECDSA_ANY);
```

**Persistent Plain Key**

<table class="classic">
  <colgroup>
    <col width="20%">
    <col width="30%">
    <col width="20%">
    <col width="30%">
  </colgroup>
  <thead>
    <tr>
      <th><strong>Key ID</strong></th>
      <th><strong>Persistence Level</strong></th>
      <th><strong>Location Indicator</strong></th>
      <th><strong>API Flow</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>&gt; 0</td>
      <td><code>PSA_KEY_PERSISTENCE_DEFAULT</code></td>
      <td>Local (0x0)</td>
      <td>
        A non-zero key ID in <code>psa_set_key_id()</code> will change the persistence level from <code>PSA_KEY_PERSISTENCE_VOLATILE</code> to <code>PSA_KEY_PERSISTENCE_DEFAULT</code>.
      </td>
    </tr>
  </tbody>
</table>

**Example**:

```sh
psa_key_attributes_t key_attr;
key_attr = psa_key_attributes_init();
psa_set_key_type(&key_attr, PSA_KEY_TYPE_ECC_KEY_PAIR(PSA_ECC_FAMILY_SECP_R1));
psa_set_key_bits(&key_attr, 256);
psa_set_key_usage_flags(&key_attr, PSA_KEY_USAGE_SIGN_HASH | PSA_KEY_USAGE_VERIFY_HASH);
psa_set_key_algorithm(&key_attr, PSA_ALG_ECDSA_ANY);
psa_set_key_id(&key_attr, 0x02);
```

**Volatile Wrapped Key**

<table class="classic">
  <colgroup>
    <col width="20%">
    <col width="30%">
    <col width="20%">
    <col width="auto">
  </colgroup>
  <thead>
    <tr>
      <th><strong>Key ID</strong></th>
      <th><strong>Persistence Level</strong></th>
      <th><strong>Location Indicator</strong></th>
      <th><strong>API Flow</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p>= 0</p></td>
      <td><p><code>PSA_KEY_PERSISTENCE_VOLATILE</code></p></td>
      <td><p>Secure (0x1)</p></td>
      <td><p>Use the <code>psa_set_key_lifetime()</code> to change the location indicator from Local to Secure (0x01).</p></td>
    </tr>
  </tbody>
</table>

**Example**:

```sh
psa_key_attributes_t key_attr;
key_attr = psa_key_attributes_init();
psa_set_key_type(&key_attr, PSA_KEY_TYPE_ECC_KEY_PAIR(PSA_ECC_FAMILY_SECP_R1));
psa_set_key_bits(&key_attr, 256);
psa_set_key_usage_flags(&key_attr, PSA_KEY_USAGE_SIGN_HASH | PSA_KEY_USAGE_VERIFY_HASH);
psa_set_key_algorithm(&key_attr, PSA_ALG_ECDSA_ANY);
psa_set_key_lifetime(&key_attr, PSA_KEY_LIFETIME_FROM_PERSISTENCE_AND_LOCATION(PSA_KEY_PERSISTENCE_VOLATILE, 0x01));
```

**Persistent Wrapped Key**

<table class="classic">
  <colgroup>
    <col width="20%">
    <col width="30%">
    <col width="20%">
    <col width="auto">
  </colgroup>
  <thead>
    <tr>
      <th><strong>Key ID</strong></th>
      <th><strong>Persistence Level</strong></th>
      <th><strong>Location Indicator</strong></th>
      <th><strong>API Flow</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p>&gt; 0</p></td>
      <td><p><code>PSA_KEY_PERSISTENCE_DEFAULT</code></p></td>
      <td><p>Local (0x0)</p></td>
      <td><p>A non-zero key ID in <code>psa_set_key_id()</code> will change the persistence level from <code>PSA_KEY_PERSISTENCE_VOLATILE</code> to <code>PSA_KEY_PERSISTENCE_DEFAULT</code>.</p></td>
    </tr>
    <tr>
      <td><p>&gt; 0</p></td>
      <td><p><code>PSA_KEY_PERSISTENCE_DEFAULT</code></p></td>
      <td><p>Secure (0x1)</p></td>
      <td><p>Use the <code>psa_set_key_lifetime()</code> to change the location indicator from Local to Secure (0x01).</p></td>
    </tr>
  </tbody>
</table>

**Example**:

```sh
psa_key_attributes_t key_attr;
key_attr = psa_key_attributes_init();
psa_set_key_type(&key_attr, PSA_KEY_TYPE_ECC_KEY_PAIR(PSA_ECC_FAMILY_SECP_R1));
psa_set_key_bits(&key_attr, 256);
psa_set_key_usage_flags(&key_attr, PSA_KEY_USAGE_SIGN_HASH | PSA_KEY_USAGE_VERIFY_HASH);
psa_set_key_algorithm(&key_attr, PSA_ALG_ECDSA_ANY);
psa_set_key_id(&key_attr, 0x02);
psa_set_key_lifetime(&key_attr, PSA_KEY_LIFETIME_FROM_PERSISTENCE_AND_LOCATION(PSA_KEY_PERSISTENCE_DEFAULT, 0x01));
```

>**Note**:
>
>- The `PSA_KEY_PERSISTENCE_DEFAULT` is equal to `PSA_KEY_LIFETIME_PERSISTENT`.
>
>- Refer to [Key Identifiers](./04-key-management-in-psa-crypto.md#key-identifiers) for details about the Key ID.
