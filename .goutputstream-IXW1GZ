#include "types.h"
#include "param.h"
#include "memlayout.h"
#include "mmu.h"
#include "proc.h"
#include "x86.h"
#include "traps.h"
#include "spinlock.h"
#include "paging.h"
#include "fs.h"
#include "defs.h"


/* Allocate eight consecutive disk blocks.
 * Save the content of the physical page in the pte
 * to the disk blocks and save the block-id into the
 * pte.
 */
void
swap_page_from_pte(pte_t *pte)
{
	cprintf("In swap_page_from_pte\n");	
	uint db;
	while((db=balloc_page(SWAPPING_DISK))==-1);
	cprintf("db allocated ");
	write_page_to_disk(SWAPPING_DISK,P2V(PTE_ADDR(*pte)),db);
	cprintf("page written to disk ");
	char *to_free = P2V(PTE_ADDR(*pte));
	set_swapped(*pte);
	clear_present(*pte);
	if(db>=(1<<20))
		panic("Error");
	*pte = (*pte & ((uint)0x0fff))|(db<<12);
	//set_swapped(*pte);
	//cprintf("%s\n",to_free);
	kfree(to_free);
	
}

/* Select a victim and swap the contents to the disk.
 */
int
swap_page(pde_t *pgdir)
{

	cprintf("In swap_page\n");	
	pte_t *victim;
	victim = select_a_victim(pgdir);
	//cprintf("%u",*victim);
	if(victim==0)
		panic("Victim not found");
	swap_page_from_pte(victim);
	return 1;	
	//panic("swap_page is not implemented");
	//return 1;
}

void allocate_page(pde_t *pgdir,uint addr)
{
	cprintf("In allocate_page\n");	
	while(allocuvm(pgdir,PGROUNDDOWN(addr),PGROUNDDOWN(addr)+PGSIZE)==0)
	{
		swap_page(pgdir);
	}
}

/* Map a physical page to the virtual address addr.
 * If the page table entry points to a swapped block
 * restore the content of the page from the swapped
 * block and free the swapped block.
 */
void
map_address(pde_t *pgdir, uint addr)
{

	//cprintf("hello rajeev");
	cprintf("In map_address\n");
	pde_t *pde;
	pte_t *pte;
	struct proc *curproc = myproc();
	if(addr>curproc->sz)
		panic("Error");
	pte = uva2pte(pgdir, addr);
	if(pte==0)
	{
		cprintf("Mapping not present ");		
		pde = &pgdir[PDX(addr)];
		if(swapped(*pde))
		{
			cprintf("Swapped ");
		}
		else
		{
			cprintf("Allocating ");			
			allocate_page(pgdir,addr);
			return;
		}
	}
	if(*pte&PTE_P)
	{
		return;
	}
	if(swapped(*pte))
	{
		cprintf("Pte Swapped ");	
		uint db = get_swapped_block_id(pte);
		allocate_page(pgdir,addr);
		clear_swap(*pte);
		read_page_from_disk(SWAPPING_DISK,P2V(PTE_ADDR(*pte)),db);
		bfree_page(SWAPPING_DISK,db);
		//set_present(*pte);
		
		
	}
	else
	{
		cprintf("Allocating ");			
		allocate_page(pgdir,addr);
		
	}
	
	//panic("done");
}

/* page fault handler */
void
handle_pgfault()
{
	cprintf("In handle_pgfault\n");	
	unsigned addr;
	struct proc *curproc = myproc();

	asm volatile ("movl %%cr2, %0 \n\t" : "=r" (addr));
	addr &= ~0xfff;
	map_address(curproc->pgdir, addr);
	asm volatile("": : :"memory");
	switchuvm(curproc);
  	//return 0;
}
