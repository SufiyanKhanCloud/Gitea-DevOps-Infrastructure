### **02-IIS-Reverse-Proxy**

**File:** `iis-optimization.md`
**Content:**

> ## IIS Reverse Proxy & Large File Optimization
> 
> 
> To support high-capacity repository management, the IIS Reverse Proxy was tuned to bypass the default 30MB request limit.
> * **Optimization:** Increased `maxAllowedContentLength` to **512MB** ( bytes).
> * **Verification:** Successfully tested with a **100 MiB** payload upload to verify the handshake between IIS and Gitea.
> * **Key Config:**
> 
> 

> ```xml
> <security>
>   <requestFiltering>
>     <requestLimits maxAllowedContentLength="536870912" />
>   </requestFiltering>
> </security>
> 
> ```
> 
> 

---
