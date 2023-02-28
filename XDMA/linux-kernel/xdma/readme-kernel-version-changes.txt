diff --git a/XDMA/linux-kernel/xdma/cdev_sgdma.c b/XDMA/linux-kernel/xdma/cdev_sgdma.c
index 2b13000..c17145f 100644
--- a/XDMA/linux-kernel/xdma/cdev_sgdma.c
+++ b/XDMA/linux-kernel/xdma/cdev_sgdma.c
@@ -105,7 +105,11 @@ static void async_io_handler(unsigned long  cb_hndl, int err)
                res = caio->res;
                res2 = caio->res2;
 #if LINUX_VERSION_CODE >= KERNEL_VERSION(4, 1, 0)
+  #if 0
                caio->iocb->ki_complete(caio->iocb, res, res2);
+  #else
+               caio->iocb->ki_complete(caio->iocb, res);
+  #endif
 #else
                aio_complete(caio->iocb, res, res2);
 #endif
@@ -120,7 +124,11 @@ skip_tran:
 
 skip_dev_lock:
 #if LINUX_VERSION_CODE >= KERNEL_VERSION(4, 1, 0)
+  #if 0
        caio->iocb->ki_complete(caio->iocb, numbytes, -EBUSY);
+  #else
+       caio->iocb->ki_complete(caio->iocb, numbytes);
+  #endif
 #else
        aio_complete(caio->iocb, numbytes, -EBUSY);
 #endif
diff --git a/XDMA/linux-kernel/xdma/libxdma.c b/XDMA/linux-kernel/xdma/libxdma.c
index 4342b79..f009207 100644
--- a/XDMA/linux-kernel/xdma/libxdma.c
+++ b/XDMA/linux-kernel/xdma/libxdma.c
@@ -2908,7 +2908,11 @@ static void transfer_destroy(struct xdma_dev *xdev, struct xdma_transfer *xfer)
                struct sg_table *sgt = xfer->sgt;
 
                if (sgt->nents) {
+#if 0
                        pci_unmap_sg(xdev->pdev, sgt->sgl, sgt->nents,
+#else
+                        dma_unmap_sg(&xdev->pdev->dev, sgt->sgl, sgt->nents,
+#endif
                                     xfer->dir);
                        sgt->nents = 0;
                }
@@ -3192,7 +3196,11 @@ ssize_t xdma_xfer_aperture(struct xdma_engine *engine, bool write, u64 ep_addr,
        }
 
        if (!dma_mapped) {
+#if 0
                sgt->nents = pci_map_sg(xdev->pdev, sgt->sgl, sgt->orig_nents,
+#else
+               sgt->nents = dma_map_sg(&xdev->pdev->dev, sgt->sgl, sgt->orig_nents,
+#endif
                                        dir);
                if (!sgt->nents) {
                        pr_info("map sgl failed, sgt 0x%p.\n", sgt);
@@ -3434,7 +3442,11 @@ ssize_t xdma_xfer_aperture(struct xdma_engine *engine, bool write, u64 ep_addr,
 
 unmap_sgl:
        if (!dma_mapped && sgt->nents) {
+#if 0
                pci_unmap_sg(xdev->pdev, sgt->sgl, sgt->orig_nents, dir);
+#else
+                dma_unmap_sg(&xdev->pdev->dev, sgt->sgl, sgt->orig_nents, dir);
+#endif
                sgt->nents = 0;
        }
 
@@ -3504,7 +3516,11 @@ ssize_t xdma_xfer_submit(void *dev_hndl, int channel, bool write, u64 ep_addr,
        }
 
        if (!dma_mapped) {
+#if 0
                nents = pci_map_sg(xdev->pdev, sg, sgt->orig_nents, dir);
+#else
+                nents = dma_map_sg(&xdev->pdev->dev, sg, sgt->orig_nents, dir);
+#endif
                if (!nents) {
                        pr_info("map sgl failed, sgt 0x%p.\n", sgt);
                        return -EIO;
@@ -3660,8 +3676,12 @@ ssize_t xdma_xfer_submit(void *dev_hndl, int channel, bool write, u64 ep_addr,
 
 unmap_sgl:
        if (!dma_mapped && sgt->nents) {
+#if 0
                pci_unmap_sg(xdev->pdev, sgt->sgl, sgt->orig_nents, dir);
-               sgt->nents = 0;
+#else
+               dma_unmap_sg(&xdev->pdev->dev, sgt->sgl, sgt->orig_nents, dir);
+#endif
+                sgt->nents = 0;
        }
 
        if (req)
@@ -3781,7 +3801,11 @@ ssize_t xdma_xfer_completion(void *cb_hndl, void *dev_hndl, int channel,
 
 unmap_sgl:
        if (!dma_mapped && sgt->nents) {
+#if 0
                pci_unmap_sg(xdev->pdev, sgt->sgl, sgt->orig_nents, dir);
+#else
+               dma_unmap_sg(&xdev->pdev->dev, sgt->sgl, sgt->orig_nents, dir);
+#endif
                sgt->nents = 0;
        }
 
@@ -3855,7 +3879,11 @@ ssize_t xdma_xfer_submit_nowait(void *cb_hndl, void *dev_hndl, int channel,
        }
 
        if (!dma_mapped) {
+#if 0
                nents = pci_map_sg(xdev->pdev, sg, sgt->orig_nents, dir);
+#else
+                nents = dma_map_sg(&xdev->pdev->dev, sg, sgt->orig_nents, dir);
+#endif
                if (!nents) {
                        pr_info("map sgl failed, sgt 0x%p.\n", sgt);
                        return -EIO;
@@ -3895,7 +3923,11 @@ ssize_t xdma_xfer_submit_nowait(void *cb_hndl, void *dev_hndl, int channel,
                        pr_info("transfer_init failed\n");
 
                        if (!dma_mapped && sgt->nents) {
+#if 0
                                pci_unmap_sg(xdev->pdev, sgt->sgl,
+#else
+                               dma_unmap_sg(&xdev->pdev->dev, sgt->sgl,
+#endif
                                                sgt->orig_nents, dir);
                                sgt->nents = 0;
                        }
@@ -3943,7 +3975,11 @@ ssize_t xdma_xfer_submit_nowait(void *cb_hndl, void *dev_hndl, int channel,
 
 unmap_sgl:
        if (!dma_mapped && sgt->nents) {
+#if 0
                pci_unmap_sg(xdev->pdev, sgt->sgl, sgt->orig_nents, dir);
+#else
+               dma_unmap_sg(&xdev->pdev->dev, sgt->sgl, sgt->orig_nents, dir);
+#endif
                sgt->nents = 0;
        }
 
@@ -4191,16 +4227,32 @@ static int set_dma_mask(struct pci_dev *pdev)
 
        dbg_init("sizeof(dma_addr_t) == %ld\n", sizeof(dma_addr_t));
        /* 64-bit addressing capability for XDMA? */
-       if (!pci_set_dma_mask(pdev, DMA_BIT_MASK(64))) {
+#if 0
+        if (!pci_set_dma_mask(pdev, DMA_BIT_MASK(64))) {
+#else
+        if (!dma_set_mask(&pdev->dev, DMA_BIT_MASK(64))) {
+#endif
                /* query for DMA transfer */
                /* @see Documentation/DMA-mapping.txt */
                dbg_init("pci_set_dma_mask()\n");
                /* use 64-bit DMA */
                dbg_init("Using a 64-bit DMA mask.\n");
+#if 0
                pci_set_consistent_dma_mask(pdev, DMA_BIT_MASK(64));
+#else
+               dma_set_coherent_mask(&pdev->dev, DMA_BIT_MASK(64));
+#endif
+#if 0
        } else if (!pci_set_dma_mask(pdev, DMA_BIT_MASK(32))) {
+#else
+       } else if (!dma_set_mask(&pdev->dev, DMA_BIT_MASK(32))) {
+#endif
                dbg_init("Could not set 64-bit DMA mask.\n");
+#if 0
                pci_set_consistent_dma_mask(pdev, DMA_BIT_MASK(32));
+#else
+               dma_set_coherent_mask(&pdev->dev, DMA_BIT_MASK(32));
+#endif
                /* use 32-bit DMA */
                dbg_init("Using a 32-bit DMA mask.\n");
        } else {