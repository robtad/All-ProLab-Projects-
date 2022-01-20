/* 
** Muhammad Abdan SYAKURA (200201147)
** Robera Tadesse GOBOSHO (190201141)
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

unsigned long    ERROR;
int count = 0;

typedef enum SKIP_TYPE     {skip, no_skip}                 SKIP_TYPE;
typedef enum RULE_2_TYPE   {new_son, split}                RULE_2_TYPE;
typedef enum LAST_POS_TYPE {last_char_in_edge, other_char} LAST_POS_TYPE;

typedef struct TreeNode
{
   struct TreeNode*   sons;
   struct TreeNode*   right_sibling;
   struct TreeNode*   left_sibling;
   struct TreeNode*   father;
   struct TreeNode*   suffix_link;
   unsigned long                 path_position;
   unsigned long                 edge_label_start;
   unsigned long                 edge_label_end;
} NODE;

NODE*    suffixless;

typedef struct TreePath
{
   unsigned long   begin;
   unsigned long   end;
} PATH;

typedef struct TreePos
{
   NODE*      node;
   unsigned long   edge_pos;
}POS;


typedef struct SuffixTree
{
   unsigned long          e;
   char*             tree_string;
   unsigned long          length;
   NODE*             root;
} SUFFIX_TREE;

SUFFIX_TREE* CreateTree(const char*   str, unsigned long length);
unsigned long FindSubstring(SUFFIX_TREE*      tree, char*    W, unsigned long P);
void PrintTree(SUFFIX_TREE* tree);
void DeleteTree(SUFFIX_TREE* tree);
unsigned long ST_SelfTest(SUFFIX_TREE* tree);

NODE* create_node(NODE* father, unsigned long start, unsigned long end, unsigned long position)
{
   NODE* node   = (NODE*)malloc(sizeof(NODE));
   if(node == 0)
   {
      printf("\nOut of memory.\n");
      exit(0);
   }
   node->sons             = 0;
   node->right_sibling    = 0;
   node->left_sibling     = 0;
   node->suffix_link      = 0;
   node->father           = father;
   node->path_position    = position;
   node->edge_label_start = start;
   node->edge_label_end   = end;
   return node;
}

NODE* find_son(SUFFIX_TREE* tree, NODE* node, char character)
{
   node = node->sons;
   while(node != 0 && tree->tree_string[node->edge_label_start] != character)
   {
      node = node->right_sibling;
   }
   return node;
}

unsigned long get_node_label_end(SUFFIX_TREE* tree, NODE* node)
{
   if(node->sons == 0)
      return tree->e;
   return node->edge_label_end;
}

unsigned long get_node_label_length(SUFFIX_TREE* tree, NODE* node)
{
   return get_node_label_end(tree, node) - node->edge_label_start + 1;
}

char is_last_char_in_edge(SUFFIX_TREE* tree, NODE* node, unsigned long edge_pos)
{
   if(edge_pos == get_node_label_length(tree,node)-1)
      return 1;
   return 0;
}

void connect_siblings(NODE* left_sib, NODE* right_sib)
{
   if(left_sib != 0)
      left_sib->right_sibling = right_sib;
   if(right_sib != 0)
      right_sib->left_sibling = left_sib;
}

NODE* apply_extension_rule_2(NODE* node, unsigned long edge_label_begin, unsigned long edge_label_end, unsigned long path_pos,unsigned long edge_pos, RULE_2_TYPE     type)
{
   NODE *new_leaf,
        *new_internal,
        *son;
   if(type == new_son)
   {
      new_leaf = create_node(node, edge_label_begin , edge_label_end, path_pos);
      son = node->sons;
      while(son->right_sibling != 0)
         son = son->right_sibling;
      connect_siblings(son, new_leaf);
      return new_leaf;
   }

   new_internal = create_node(
                      node->father,
                      node->edge_label_start,
                      node->edge_label_start+edge_pos,
                      node->path_position);
   node->edge_label_start += edge_pos+1;

   new_leaf = create_node(
                      new_internal,
                      edge_label_begin,
                      edge_label_end,
                      path_pos);

   connect_siblings(node->left_sibling, new_internal);
   connect_siblings(new_internal, node->right_sibling);
   node->left_sibling = 0;

   if(new_internal->father->sons == node)
      new_internal->father->sons = new_internal;

   new_internal->sons = node;
   node->father = new_internal;
   connect_siblings(node, new_leaf);

   return new_internal;
}

NODE* trace_single_edge(SUFFIX_TREE* tree, NODE* node, PATH str, unsigned long* edge_pos, unsigned long* chars_found, SKIP_TYPE type, int* search_done)
{
   NODE*      cont_node;
   unsigned long   length,str_len;

   *search_done = 1;
   *edge_pos    = 0;

   cont_node = find_son(tree, node, tree->tree_string[str.begin]);
   if(cont_node == 0)
   {
      *edge_pos = get_node_label_length(tree,node)-1;
      *chars_found = 0;
      return node;
   }

   node    = cont_node;
   length  = get_node_label_length(tree,node);
   str_len = str.end - str.begin + 1;

   if(type == skip)
   {
      if(length <= str_len)
      {
         (*chars_found)   = length;
         (*edge_pos)      = length-1;
         if(length < str_len)
            *search_done  = 0;
      }
      else
      {
         (*chars_found)   = str_len;
         (*edge_pos)      = str_len-1;
      }

      return node;
   }
   else
   {
      if(str_len < length)
         length = str_len;

      for(*edge_pos=1, *chars_found=1; *edge_pos<length; (*chars_found)++,(*edge_pos)++)
      {
         if(tree->tree_string[node->edge_label_start+*edge_pos] != tree->tree_string[str.begin+*edge_pos])
         {
            (*edge_pos)--;
            return node;
         }
      }
   }
   (*edge_pos)--;

   if((*chars_found) < str_len)
      *search_done = 0;

   return node;
}

NODE* trace_string(SUFFIX_TREE* tree, NODE* node, PATH str, unsigned long* edge_pos, unsigned long* chars_found, SKIP_TYPE type)
{
   int      search_done = 0;
   unsigned long edge_chars_found;

   *chars_found = 0;

   while(search_done == 0)
   {
      *edge_pos        = 0;
      edge_chars_found = 0;
      node = trace_single_edge(tree, node, str, edge_pos, &edge_chars_found, type, &search_done);
      str.begin       += edge_chars_found;
      *chars_found    += edge_chars_found;
   }
   return node;
}

void findSon(NODE* node){

        while(node->sons != 0){
            if(node->sons->sons != 0){
                findSon(node->sons);
            }
            else
                count++;
            node->sons = node->sons->right_sibling;
            
        }
        
}

unsigned long FindSubstring(SUFFIX_TREE* tree, char*  W,unsigned long P)
{
   NODE* node   = find_son(tree, tree->root, W[0]);
   unsigned long k,j = 0, node_label_end;

   while(node!=0)
   {
      k=node->edge_label_start;
      node_label_end = get_node_label_end(tree,node);

      while(j<P && k<=node_label_end && tree->tree_string[k] == W[j])
      {
         j++;
         k++;
      }

      if(j == P)
      {
         findSon(node);
         if(count == 0){
             count++;
         }
         return node->path_position;
      }
      else if(k > node_label_end)
         node = find_son(tree, node, W[j]);
      else
      {
         return ERROR;
      }
   }
   return ERROR;
}

void follow_suffix_link(SUFFIX_TREE* tree, POS* pos)
{
   PATH      gama;
   unsigned long  chars_found = 0;

   if(pos->node == tree->root)
   {
      return;
   }
   if(pos->node->suffix_link == 0 || is_last_char_in_edge(tree,pos->node,pos->edge_pos) == 0)
   {
      if(pos->node->father == tree->root)
      {
         pos->node = tree->root;
         return;
      }
      gama.begin      = pos->node->edge_label_start;
      gama.end      = pos->node->edge_label_start + pos->edge_pos;
      pos->node      = pos->node->father->suffix_link;
      pos->node      = trace_string(tree, pos->node, gama, &(pos->edge_pos), &chars_found, skip);
   }
   else
   {
      pos->node      = pos->node->suffix_link;
      pos->edge_pos   = get_node_label_length(tree,pos->node)-1;
   }
}

void create_suffix_link(NODE* node, NODE* link)
{
   node->suffix_link = link;
}

void SEA(SUFFIX_TREE* tree, POS* pos,PATH str,unsigned long* rule_applied, char after_rule_3)
{
   unsigned long   chars_found = 0 , path_pos = str.begin;
   NODE*      tmp;

   if(after_rule_3 == 0)
      follow_suffix_link(tree, pos);

   if(pos->node == tree->root)
   {
      pos->node = trace_string(tree, tree->root, str, &(pos->edge_pos), &chars_found, no_skip);
   }
   else
   {
      str.begin = str.end;
      chars_found = 0;

      if(is_last_char_in_edge(tree,pos->node,pos->edge_pos))
      {
         tmp = find_son(tree, pos->node, tree->tree_string[str.end]);
         if(tmp != 0)
         {
            pos->node      = tmp;
            pos->edge_pos   = 0;
            chars_found      = 1;
         }
      }
      else
      {
         if(tree->tree_string[pos->node->edge_label_start+pos->edge_pos+1] == tree->tree_string[str.end])
         {
            pos->edge_pos++;
            chars_found   = 1;
         }
      }
   }
   if(chars_found == str.end - str.begin + 1)
   {
      *rule_applied = 3;
      if(suffixless != 0)
      {
         create_suffix_link(suffixless, pos->node->father);
         suffixless = 0;
      }
      return;
   }
   if(is_last_char_in_edge(tree,pos->node,pos->edge_pos) || pos->node == tree->root)
   {
      if(pos->node->sons != 0)
      {
         apply_extension_rule_2(pos->node, str.begin+chars_found, str.end, path_pos, 0, new_son);
         *rule_applied = 2;
         if(suffixless != 0)
         {
            create_suffix_link(suffixless, pos->node);
            suffixless = 0;
         }
      }
   }
   else
   {
      tmp = apply_extension_rule_2(pos->node, str.begin+chars_found, str.end, path_pos, pos->edge_pos, split);
      if(suffixless != 0)
         create_suffix_link(suffixless, tmp);
      if(get_node_label_length(tree,tmp) == 1 && tmp->father == tree->root)
      {
         tmp->suffix_link = tree->root;
         suffixless = 0;
      }
      else
         suffixless = tmp;

      pos->node = tmp;
      *rule_applied = 2;
   }
}

void SPA(SUFFIX_TREE* tree, POS* pos, unsigned long phase, unsigned long* extension, char* repeated_extension)
{
   unsigned long   rule_applied = 0;
   PATH       str;

   tree->e = phase+1;

   while(*extension <= phase+1)
   {
      str.begin       = *extension;
      str.end         = phase+1;

      SEA(tree, pos, str, &rule_applied, *repeated_extension);

      if(rule_applied == 3)
      {
         *repeated_extension = 1;
         break;
      }
      *repeated_extension = 0;
      (*extension)++;
   }
   return;
}

SUFFIX_TREE* CreateTree(const char* str, unsigned long length)
{
   SUFFIX_TREE*  tree;
   unsigned long      phase , extension;
   char          repeated_extension = 0;
   POS           pos;

   if(str == 0)
      return 0;

   tree = malloc(sizeof(SUFFIX_TREE));
   if(tree == 0)
   {
      printf("\nOut of memory.\n");
      exit(0);
   }

   tree->length         = length+1;
   ERROR            = length+10;

   tree->tree_string = malloc((tree->length+1)*sizeof(char));
   if(tree->tree_string == 0)
   {
      printf("\nOut of memory.\n");
      exit(0);
   }

   memcpy(tree->tree_string+sizeof(char),str,length*sizeof(char));
   tree->tree_string[tree->length] = '$';

   tree->root            = create_node(0, 0, 0, 0);
   tree->root->suffix_link = 0;

   extension = 2;
   phase = 2;

   tree->root->sons = create_node(tree->root, 1, tree->length, 1);
   suffixless       = 0;
   pos.node         = tree->root;
   pos.edge_pos     = 0;

   for(; phase < tree->length; phase++)
   {
      SPA(tree, &pos, phase, &extension, &repeated_extension);
   }
   return tree;
}

void DeleteSubTree(NODE* node)
{
   if(node == 0)
      return;
   if(node->right_sibling!=0)
      DeleteSubTree(node->right_sibling);
   if(node->sons!=0)
      DeleteSubTree(node->sons);
   free(node);
}

void DeleteTree(SUFFIX_TREE* tree)
{
   if(tree == 0)
      return;
   DeleteSubTree(tree->root);
   free(tree);
}

void PrintNode(SUFFIX_TREE* tree, NODE* node1, long depth)
{
   NODE* node2 = node1->sons;
   long  d = depth , start = node1->edge_label_start , end;
   end     = get_node_label_end(tree, node1);

   if(depth>0)
   {
      while(d>1)
      {
         printf("| ");
         d--;
      }
      printf("+");
      while(start<=end)
      {
         printf("%c",tree->tree_string[start]);
         start++;
      }
      printf("\n");
   }
   while(node2!=0)
   {
      PrintNode(tree,node2, depth+1);
      node2 = node2->right_sibling;
   }
}

void PrintTree(SUFFIX_TREE* tree)
{
   printf("\nroot\n+\n");
   PrintNode(tree, tree->root, 0);
}
void birNodeYaz(SUFFIX_TREE* tree, NODE* node1){
    long start = node1->edge_label_start , end;
    end  = get_node_label_end(tree, node1);
    while(start<=end)
    {
        printf("%c",tree->tree_string[start]);
        start++;
    }
    printf("\n");
}
void birDalYaz(SUFFIX_TREE* tree, NODE* node1){
    NODE* node2 = node1->sons;
    long start = node1->sons->edge_label_start , end;
    end  = get_node_label_end(tree, node1->sons);
    while(start<=end)
    {
        printf("%c",tree->tree_string[start]);
        start++;
    }
    printf("\n");
    node2 = node2->right_sibling;
    while(node2 != 0){
        birNodeYaz(tree, node2);
        node2 = node2->right_sibling;
    }
}
//4. soru buradan basliyor
int count2 = 0, temp = 0, enFazla = 0;

void kacTane(NODE* node){

        while(node->sons != 0){
            if(node->sons->sons != 0){
                kacTane(node->sons);
                }
            else
                temp++;
            node->sons = node->sons->right_sibling;
        }
    
    if(temp==0)
        temp++;
    
}
void hangiNode(SUFFIX_TREE* tree, NODE* node1){
    long start = node1->edge_label_start , end;
    end  = node1->edge_label_end;
    while(start<=end)
    {
        printf("%c",tree->tree_string[start]);
        start++;
    }
    //printf("\n");
}

void enFazlaTekrar(SUFFIX_TREE* tree,NODE* node1){
    NODE* node2 = node1->sons;
    while(node2 != 0){
        kacTane(node2);
        //printf("%d\n", temp);
        if(temp>enFazla){
            enFazla = temp;
            count2++;
        }
        temp = 0;
        node2 = node2->right_sibling;
    }

    //printf("%d, %d\n", enFazla, count2);
    node1 = tree->root->sons;
    for(int i=0; i<count2-1; i++){
        node1 = node1->right_sibling;
    }
   printf("\n2. EN COK TEKRAR EDEN : \"");
    hangiNode(tree, node1);
    printf("\" ALTKATARDIR. \"%d\" KEZ TEKRAR ETMEKTEDIR.\n\n", enFazla);
}
//4. soru bitisi

int n=0, count3=0, sonCounter=0;
unsigned long uzunluk = 0, temp2 = 0, enUzun=0;
//3. soru baslangici
void kacTane2(SUFFIX_TREE* tree, NODE* node){

    if(node->sons != 0){  
        uzunluk += node->edge_label_end - node->edge_label_start + 1;
        n++;
        while(node->sons != 0){
            if(node->sons->sons != 0){
                kacTane2(tree, node->sons);
            }
            node->sons = node->sons->right_sibling;
        }
    }
    else
        n++;
}

void printLRS(SUFFIX_TREE* tree, NODE* node1){
    long start = node1->edge_label_start , end;
    end  = enUzun+1;
    while(start<=end)
    {
        printf("%c",tree->tree_string[start]);
        start++;
    }
}    
void sonCount(NODE* node){
  while(node != 0){
        if(node->sons != 0)
            sonCount(node->sons);
        else
            sonCounter++;
        node = node->right_sibling;
   }
}
void FindLRS(SUFFIX_TREE* tree, NODE* node1){
    NODE* node2 = node1->sons;
    while(node2 != 0){
        kacTane2(tree, node2);
        if(uzunluk>enUzun){
            enUzun = uzunluk; 
            count3++;
        }
        uzunluk = 0;
    
        node2 = node2->right_sibling;
    }
    
    node1 = tree->root->sons;
    for(int i=0; i<count3; i++){
        node1 = node1->right_sibling;
    }
    //sonCount(node1);
    printf("3. TEKRAR EDEN EN UZUN KATAR : ");
    printLRS(tree, node1);
    printf(". %d KEZ TEKRAR ETMEKTEDIR.\n", sonCounter);
}


int main(int argc, char* argv[])
{
   unsigned char *filename;
	FILE* file;
	unsigned long i, len = 0;
   char *str = NULL;

   filename = (unsigned char*)argv[1];
	file = fopen((const char*)filename,"r");
	if(file == NULL || argv[1] == NULL)
	{
		printf("\ncan't open file.\n");
		return(0);
	}
	fseek(file, 0, SEEK_END);
	len = ftell(file);
	fseek(file, 0, SEEK_SET);
	str = malloc(len*sizeof(unsigned char));
	if(str == 0)
	{
		printf("\nOut of memory.\n");
		exit(0);
	}
	fread(str, sizeof(char), len, file);
	unsigned long position;

   SUFFIX_TREE* tree = CreateTree(str, len);

	PrintTree(tree);

   if(argv[2]!=NULL){ //if we don't put second parametre it won't be segmentation fault
      position = FindSubstring(tree, (unsigned char*)(argv[2]), strlen(argv[2]));

      if(position == ERROR)
			printf("\nSONUC: \"%s\", %s KATARININ ALTKATARI DEGILDIR.\n\n", argv[2], str);
		else
            printf("\n1. \"%s\" KATARI ICINDE \"%s\" ALTKATARI %ld BASLANGIC POZISYONUNDA BULUNDU. %d KEZ TEKRAR ETMEKTEDIR.\n", str, argv[2], position, count);
   }

   enFazlaTekrar(tree, tree->root);
    
   tree = CreateTree(str, len);

   FindLRS(tree, tree->root);

	DeleteTree(tree);

	return 0;
}