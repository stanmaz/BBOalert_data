### Definitions
 
# Calculate Short Suit Points for North (shortSuitPoints)
    v1 = shape(north, any 0xxx) ? 5 : 0  // allow for 2 voids
    v2 = shape(north, any 00xx) ? 5 : 0
    s1 = shape(north, any 1xxx) ? 3 : 0 // allow for 2 singletons
    s2 = shape(north, any 11xx) ? 3 : 0
    d1 = shape(north, any 2xxx) ? 1 : 0 // allow for 3 doubletons
    d2 = shape(north, any 22xx) ? 1 : 0
    d3 = shape(north, any 222x) ? 1 : 0
    shortSuitPoints = v1+v2+s1+s2+d1+d2+d3
    supportPoints = shortSuitPoints + hcp(north)

# Calculate length points for South (lengthPoints)
    lp1 = spades(south)>4 ? spades(south)-4 : 0
    lp2 = hearts(south)>4 ? hearts(south)-4 : 0
    lp3 = diamonds(south)>4 ? diamonds(south)-4 : 0
    lp4 = clubs(south)>4 ? clubs(south)-4 : 0
    lengthPoints = lp1 + lp2 + lp3 + lp4

# Calculate doubleton honor NT downgrade(s) for South -- 2 cards, 1 honor, not the Ace
    S2H = spades(south)==2 and top4(south,spades)==1 and not hascard(south,AS) ? 1 : 0
    H2H = hearts(south)==2 and top4(south,hearts)==1 and not hascard(south,AH) ? 1 : 0
    D2H = diamonds(south)==2 and top4(south,diamonds)==1 and not hascard(south,AD) ? 1 : 0
    C2H = clubs(south)==2 and top4(south,clubs)==1 and not hascard(south,AC) ? 1 : 0
    ntDownGrade = S2H + H2H + D2H + C2H

# Define notrump points for south (ntPoints)
    ntPoints = hcp(south) + lengthPoints - ntDownGrade

# Define suit points for south (suitPoints)
    suitPoints = hcp(south) + lengthPoints

# Define robot notrump shape and exclude any 5card 
    ntShape = shape(south, any 4333 +any 4432 +any 5332 +any 5422 -5xxx -x5xx)

# Define ntPoint ranges
    oneNT = ntShape and ntPoints>14 and ntPoints<18
    twoNT = ntShape and ntPoints>19 and ntPoints<22
    weakNT = ntShape and ntPoints>10 and ntPoints<15
    overcallNT = ntShape and ntPoints>14 and ntPoints<19  // 15-18

# Define Game Force 2C
    gameForce2C = hcp(south)>22

### Predict South's opening BID
    P1 = gameForce2C
    P2 = (twoNT or oneNT) and not P1

# Predict South's Opening suit
    s = spades(south)
    h = hearts(south)
    d = diamonds(south)
    c = clubs(south)
    oS = s>4 and s>=h and s>=d and s>=c and not (P1 or P2)
    oH = not oS and h>4 and h>=d and h>=c not (P1 or P2)
    oD = not (oS or oH) and ((d>3 and d>=c) or c<3) not (P1 or P2)
    oC = not (oS or oH or oD) and not (P1 or P2)
    openingSuit = (oS or oH or oD or oC)

# Define opening Major and opening Minor
    oneSpade = oS
    oneHeart = oH
    oneDiamond = oD
    oneClub = oC
    oneMajor = (oS or oH)
    oneMinor = (oC or oD)

# Define 3+ card Fits for south
    sFit3 = oneSpade and spades(north)>2
    hFit3 = oneHeart and hearts(north)>2

# Define 4+ card fits for south
    sFit4 = oneSpade and spades(north)>3
    hFit4 = oneHeart and hearts(north)>3
    majorFit4 = sFit4 or hFit4

# Define Good suits -- 5+ cards with 2 of the top 3
    gS = spades(south)>4 and top3(south,spades)>1
    gH = hearts(south)>4 and top3(south,hearts)>1
    gD = diamonds(south)>4 and top3(south,diamonds)>1
    gC = clubs(south)>4 and top3(south,clubs)>1

# Define Strong suits -- 5+ cards with 3 of the top 4
# Define Solid suits -- 5 cards with 4 of the top 4 or 6+ cards with 3 of the top 3  

# Define pesky opps e/w distributions and HCP.  We don’t want them mucking up our auctions
    calmEast = shape(east,xxxx -any 8xxx -any 7xxx -any 6xxx -any 55xx)
    calmWest = shape(west,xxxx -any 8xxx -any 7xxx -any 6xxx -any 55xx)
    calmOpps= calmEast and calmWest


# Define East weak 2 bids
    w2S = spades(east)==6 and top5(east,spades)>2 and hcp(east,spades)>4 and hearts(east)<4 and spades(south)<3 and spades(west)<3
    w2H = hearts(east)==6 and top5(east,hearts)>2 and hcp(east,hearts)>4 and spades(east)<4 and hearts(south)<3 and hearts(west)<3
    w2D = diamonds(east)==6 and top5(east,diamonds)>2 and hcp(east,diamonds)>4 and spades(east)<4 and hearts(east)<4 and diamonds(south)<3
    eastWeak2 = (w2S or w2H or w2D) and hcp(east)>5 and hcp(east)<10 and shape(east,any 6430 +any 6421 +any 6331 +any 6322)  // should use east's lp rather than hcp

### All of the following is for calculating the suit ranks for North opener, East overall, and South new suit ###

# Predict North's opening suit
    sN = spades(north)
    hN = hearts(north)
    dN = diamonds(north)
    cN = clubs(north)
    nS = sN>4 and sN>=hN and sN>=dN and sN>=cN
    nH = not nS and hN>4 and hN>=dN and hN>=cN
    nD = not nS and not nH and ((dN>3 and dN>=cN) or cN<3)
    nC = not nS and not nH and not nD

# Calculate North's Rank
    nRS = nS ? 4 : 0
    nRH = nH ? 3 : 0
    nRD = nD ? 2 : 0
    nRC = nC ? 1 : 0
    northRank = nRS+nRH+nRD+nRC   // all except one are zero

# East's longest suit for overcall
    sE = spades(east)
    hE = hearts(east)
    dE = diamonds(east)
    cE = clubs(east)
    eS = sE>=hE and sE>=dE and sE>=cE
    eH = not eS and hE>=dE and hE>=cE
    eD = not eS and not eH and dE>=cE
    eC = not eS and not eH and not eD

# Calculate East's Rank
    eRS = eS ? 4 : 0
    eRH = eH ? 3 : 0
    eRD = eD ? 2 : 0
    eRC = eC ? 1 : 0
    eastRank = eRS+eRH+eRD+eRC

# South's longest suit for responding in a new suit at the 2-level
    s = spades(south)
    h = hearts(south)
    d = diamonds(south)
    c = clubs(south)
    sS = s>=h and s>=d and s>=c
    sH = not sS and h>=d and h>=c
    sD = not sS and not sH and d>=c
    sC = not sS and not sH and not sD

# Calculate South's Rank
    sRS = sS ? 4 : 0
    sRH = sH ? 3 : 0
    sRD = sD ? 2 : 0
    sRC = sC ? 1 : 0
    southRank = sRS + sRH + sRD + sRC

# Requirement for a Free Bid, Negative or Otherwise
    (northRank > eastRank) or (eastRank > southRank)
    
