/*
 * mesal az graohi k 2 bakhshi nis
0 1 0 1 0 0
1 0 1 0 0 0
0 1 0 1 0 0
1 0 1 0 1 1
0 0 0 1 0 1
0 0 0 1 1 0

mesale 2 bakhshi
0 1 0 1 0 0 0
1 0 1 0 0 0 1
0 1 0 1 1 0 0
1 0 1 0 0 1 1
0 0 1 0 0 1 0
0 0 0 1 1 0 0
0 1 0 1 0 0 0

*/
import java.util.*;
public class Matching_DS {
	
	public static void main(String[]args){
		Scanner scan = new Scanner(System.in);
		
		// read Matrix
		System.out.println("Enter the number of vertices") ;
		int n = scan.nextInt() ;
		System.out.println("Enter the Matrice") ;
		int[][] matrice = new int[n][n] ;
		for(int i=0 ; i<n ; i++)
			for(int j=0 ; j<n ; j++)
				matrice[i][j] = scan.nextInt();
		
		//we need "best" for finding parts of graph
		int[] best = new int[n] ;
		Arrays.fill(best, Integer.MAX_VALUE);
		
		// check if our graph is bipartible or not
		if(bipartible(matrice,n,best)){
			//....&&....
			System.out.print("bakhsh-haye graph :") ;
			
			// *find the parts
			LinkedList<Integer> first = new LinkedList<>() ;
			LinkedList<Integer> second = new LinkedList<>() ;	
			for(int i=0;i<n;i++){
				if(best[i]%2==0){
					first.add(i) ;
				}
				else
					second.add(i) ;
			}
			//.......&&...
			System.out.println("\n"+first) ;
			System.out.println(second) ;
			
			int[][] matrix = new int[n+2][n+2] ;
			make_matrice (matrice, matrix, first, second) ;
			
			int res = Max_flow(matrix,first,second) ;
			//......&&...
			System.out.println("\nmax matching : "+res) ;
			if(res==first.size()){
				System.out.println("Complet Match") ;
			}
		}
		else{
			System.out.println("Your matrice is not bapartible !!") ;
		}
	}
	
	//.......................................................................................
	// *check if a graph is bipatrible or not
	public static boolean bipartible(int[][] matrice ,int n,int[] best){
		int[] prev = new int[n];
		Arrays.fill(prev, -1);
		
		boolean[] visited = new boolean[n];

		LinkedList<Integer> queue = new LinkedList<Integer>() ;
		
		//integer startVertice = scan.nextInt();
		visited[0] = true ;
		best[0] = 0 ;
		queue.addFirst(0) ;
		
		while(!queue.isEmpty())
		{
			int at = queue.removeLast();
			
			for(int i=0 ; i<n ; i++){
				//check if it is bipartible or not
				if(visited[i] && matrice[at][i]==1){
					if(best[i]%2==best[at]%2){
						return false ;
					}
				}
				if(!visited[i] && matrice[at][i]==1)
				{
					best[i] = best[at]+1 ;
					visited[i] = true ;
					prev[i] = at ;
					queue.addFirst(i) ;
				}
			}
		}
		return true ;
	}
	//.............................................................................
	// add a sink and source to the graph and make a network 
	public static void make_matrice(int[][] matrice, int[][] matrix, LinkedList<Integer> first, LinkedList<Integer> second){
		int n = matrice.length ;
		
		for(int i=0;i<n+2;i++){
			matrix[0][i] = 0 ;
			matrix[i][0] = 0 ;
			matrix[0][n-1] = 0 ;
			matrix[n-1][0] = 0 ;
		}
		for(int i=0 ; i<first.size() ; i++){
			matrix[0][first.get(i)+1] = 1 ;
		}
		for(int i=0 ; i<second.size() ; i++){
			matrix[second.get(i)+1][n+1] = 1 ;
		}
		for(int i=0 ; i<n ; i++){
			for(int j=0 ; j<n ; j++){
				if( matrice[i][j]==1){
					//mikhahim b in vasile graph ro jahatdar b samte sink konim
					if(member(i,first) && member(j,second))
						matrix[i+1][j+1] = 1 ;
				}
				else{
					matrix[i+1][j+1] = 0 ;
				}
			}
		}
	}
	//.............................................................................
	// check if a number is a member of the linked list !!
	public static boolean member(int k, LinkedList<Integer> list){
		for(int i=0 ; i<list.size() ; i++){
			if(list.get(i)==k){
				return true ;
			}
		}
		return false ;
	}
	
	//.............................................................................
	public static int Max_flow(int[][] matrix, LinkedList<Integer> first, LinkedList<Integer> second) {
		int n = matrix.length ;
		int[][] flow = new int[n][n] ;
		for(int i=0;i<n;i++){
			for(int j=0;j<n;j++){
				flow[i][j] = 0 ;
			}
		}
		
		// tashkile geraphe residual network
		int[][] ResidualNetwork = new int[n][n] ;		
		for(int i=0 ; i<n ; i++){
			for(int j=0 ; j<n ; j++){
				 if(matrix[i][j]==1){
					 if(matrix[i][j]>flow[i][j]){
						 ResidualNetwork[i][j] = 1 ;
					 }
					 if(matrix[j][i]==1){
						 if(flow[j][i]>0){
							 ResidualNetwork[i][j]=0 ;
						 }
					 }
				 }
			}
		}
		
		// bfs bezanim ru residual network bebinim s-t path dare ya na 
		int result = 0, path_capacity ;
		System.out.print("\n matches ::") ;
		while(true){
			path_capacity = bfs(n, matrix, 0, n-1) ;
			if(path_capacity==0){
				break ;
			}
			else
				result += path_capacity ;
		}
		return(result) ;
	}
	///..................................
	public static int bfs(int n, int[][] capacity, int source, int sink) {
		LinkedList<Integer> queue = new LinkedList<>() ;
		boolean[] visited = new boolean[n] ;
		
		queue.add(source) ;
		visited[source] = true ;
		int[] from = new int[n] ;
		Arrays.fill(from, -1) ;
		int at ;
		
		while(!queue.isEmpty()){
			
			at = queue.removeLast() ;
			for(int i=0 ; i<n ; i++){
				if(capacity[at][i]==1){
					if(!visited[i]){
						queue.add(i) ;
						visited[i] = true ;
						from[i] = at ;
						if(i==sink){
							break ;
						}
					}
				}
			}
		}
		
		//compute the path capacity
		at = sink ;
		int path_cap = Integer.MAX_VALUE, prev ;
		while(from[at]>-1){
			prev = from[at] ;
			// to print matches...
			if(prev!=0)
				System.out.print(" "+(prev-1)) ;
			else
				System.out.print(" ,") ;
			
			path_cap = Math.min(path_cap, capacity[prev][at]) ;
			at = prev ;
		}
		
			
		// we update the residual network; if no path is found the while   	
		//loop will not be entered
		at = sink ;
		while(from[at]>-1){
			prev = from[at] ;
			capacity[prev][at] -= path_cap ;
			capacity[at][prev] += path_cap ;
			at = prev ;
		}
		
		if(path_cap==Integer.MAX_VALUE){
			return 0 ;
		}
		else
			return (path_cap) ;
		
	}
}