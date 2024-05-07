# Proton experimental bleeding edge with unsafe Apex Legends fix
Apex Legends season 21 update broke the game on Linux. ~~This is a temporary repository with a really shitty and unsafe fix for the issue until the game gets fixed.~~ The problem was fixed in Wine/Proton. There was an unimplemented cryptography feature in Proton. The fix is in the latest Proton Experimental bleeding edge.

I changed a function in wine/dlls/ncrypt/main.c like this:

```diff
SECURITY_STATUS WINAPI NCryptVerifySignature(NCRYPT_KEY_HANDLE handle, void *padding, BYTE *hash, DWORD hash_size,
                                             BYTE *signature, DWORD signature_size, DWORD flags)
{
    struct object *key_object = (struct object *)handle;

    TRACE("(%#Ix, %p, %p, %lu, %p, %lu, %#lx)\n", handle, padding, hash, hash_size, signature,
          signature_size, flags);

    if (!hash_size || !signature_size) return NTE_INVALID_PARAMETER;
    if (!hash || !signature) return HRESULT_FROM_WIN32(RPC_X_NULL_REF_POINTER);
    if (!handle || key_object->type != KEY) return NTE_INVALID_HANDLE;

    if (key_object->key.algid < RSA)
    {
        FIXME("Symmetric keys not supported.\n");
        return NTE_NOT_SUPPORTED;
    }

-    return map_ntstatus(BCryptVerifySignature(key_object->key.bcrypt_key, padding, hash, hash_size, signature, signature_size, flags));
+    return ERROR_SUCCESS;
}
```
